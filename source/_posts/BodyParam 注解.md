---
title: BodyParam 注解
date: 2020-07-21
categories: [开发技术, Java, 工具设计]
tags: [工具, 注解, 实现]
---
- [需求概述](#需求概述)
- [背景说明](#背景说明)
- [设计方案](#设计方案)
- [具体实现](#具体实现)
- [遇到问题](#遇到问题)
  - [问题](#问题)
  - [解法](#解法)
- [改进实现](#改进实现)
- [参考文章](#参考文章)

# 需求概述
在控制器方法签名中，直接通过变量名获取 POST 请求体中的参数。

# 背景说明
在 SpringMVC 框架下，控制器方法想获取 POST 请求体，一般只能使用 @RequestBody 注解，常规使用方式如下：
``` java
    // 方式 1：注解 + Map
    // 注意：Map 实际从请求 Json 体转换而来，其中各键对应 Json 体中各字段。
    @PostMapping(...)
    public Object create(@RequestBody Map<String, Object> map) {...}

    // 方式 2：注解 + 对象
    // 注意：这里 Params 是专门针对请求参数自定义的类型，内部各成员对应了请求 Json 体中各字段。
    @PostMapping(...)
    public Object create(@RequestBody Params params) {...}
```

上述两种方式相对都比较繁琐：
* 第一种方式从 Map 中利用字符串类型的键获取值对象，并需要主动转换为目标格式，被动跳过编译期检查而带来隐患。
* 第二种方式需要专门定义请求参数类，当统一控制器中各方法的参数个数、类型、名称各不相同时需要创建很多参数类，使得后期维护成本较高。

有没有办法能直接从请求 Json 体中获取某一或某几字段的值，并自动转换为目标格式呢？使用方式类似 @RequestParam：
``` java
    @GetMapping(...)
    public Object query(@RequestParam String p1, 
                        @RequestParam Object p2, 
                        @RequestParam Params p3) {...}
```
可惜 @RequestParam 只能应用在 URL 参数或 form 形式的请求体上，对 Json 格式的请求体无能为力。

# 设计方案
* 注解：以 @RequestParam 为蓝本，自定义注解 @BodyParam，它同样应用在方法参数上，代表该参数是从请求体中抽取、解析得到的。
* 解析：定义注解解析器 BodyParamResolver，该解析器继承自 HandlerMethodArgumentResolver，Spring 框架会在传入方法参数前、解析方法参数时调用它，利用它来解析得到期望的参数值。

# 具体实现
> BodyParam.java
``` java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface BodyParam {
    String value() default "";
}
```

> BodyParamResolver.java
``` java
public class BodyParamResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterAnnotation(BodyParam.class) != null;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory)
            throws IOException {
        String name = getParamName(parameter);
        String body = readNativeBody(webRequest);
        return parse(parameter, name, body);
    }

    private String getParamName(MethodParameter parameter) {
        String name = parameter.getParameterAnnotation(BodyParam.class).value();
        if (StringUtils.isBlank(name)) {
            name = parameter.getParameterName();
        }
        return name;
    }

    private String readNativeBody(NativeWebRequest webRequest)
            throws IOException {
        HttpServletRequest req = webRequest.getNativeRequest(
                HttpServletRequest.class);

        // 每次获取的流对象都不同，所以要使用变量保存，避免在循环中重复调用 get 方法。
        // 至于为什么流对象每次会变，可参看后面遇到的问题及解法。
        BufferedReader r = req.getReader();
        StringBuilder sb = new StringBuilder();
        char[] buffer = new char[1024];
        int len;
        while ((len = r.read(buffer)) != -1) {
            sb.append(buffer, 0, len);
        }
        return sb.toString();
    }

    private Object parse(MethodParameter parameter, String name, String body)
            throws IOException {
        Class<?> type = parameter.getParameterType();
        JsonNode node = JsonUtils.parse(body).get(name);
        if (node == null || node instanceof NullNode) {
            return null;
        }
        return JsonUtils.parse(node, type);
    }
}
```

> applicationContext-mvc.xml
``` xml
    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean class="com.sankuai.hulk.nodemanager.api.standard.BodyParamResolver" />
        </mvc:argument-resolvers>
    </mvc:annotation-driven>
```

# 遇到问题
## 问题
* 每个参数解析时都要重新读取请求体、解析 Json，但是 HttpServletRequest 默认数据流在首次输出结束后即被关闭，也就是只有第一个参数会被解析出来，其他参数都没法获取到。
## 解法
1. 创建可以多次读取请求体的新请求类型 MultiReadHttpServletRequest，在初始化该请求时即从 HttpServletRequest 读取一次请求体并保存到本地，以后外部服务每次要获取数据流时都返回一个新的流对象。
2. 为了应用新请求类型，需要定义一个 Filter 负责将原始 HttpServletRequest 替换为包装后的新请求，该 Filter 优先级较低，要在其他 Filter 完成对请求的修饰、处理后再执行。

# 改进实现
> MultiReadHttpServletRequest.java
``` java
public class MultiReadHttpServletRequest extends HttpServletRequestWrapper {
    private final byte[] body;

    public MultiReadHttpServletRequest(HttpServletRequest req)
            throws IOException {
        super(req);
        this.body = IOUtils.toByteArray(req.getInputStream());
    }

    /**
     * 注意：不要缓存输入流，每次返回新的流对象，多次获取才不会冲突。
     */
    @Override
    public ServletInputStream getInputStream() {
        return new InputStream(body);
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream(),
                getCharacterEncoding()));
    }

    private static class InputStream extends ServletInputStream {
        private final ByteArrayInputStream inner;

        public InputStream(byte[] bytes) {
            inner = new ByteArrayInputStream(bytes);
        }

        @Override
        public int read() {
            return inner.read();
        }
    }
}
```

> MultiReadRequestFilter.java
``` java
public class MultiReadRequestFilter extends GenericFilterBean {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain)
            throws IOException, ServletException {
        chain.doFilter(new MultiReadHttpServletRequest(
                (HttpServletRequest) request), response);
    }
}
```

> web.xml
``` xml
    <filter>
        <filter-name>MultiReadRequestFilter</filter-name>
        <filter-class>com.sankuai.hulk.nodemanager.api.standard.MultiReadRequestFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MultiReadRequestFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

# 参考文章
* [实现 HttpServletRequest.getInputStream 多次读取](https://javacfox.github.io/2019/06/28/%E5%AE%9E%E7%8E%B0HttpServletRequest-getInputStream%E5%A4%9A%E6%AC%A1%E8%AF%BB%E5%8F%96/)
* [Http Servlet request lose params from POST body after read it once](https://stackoverflow.com/questions/10210645/http-servlet-request-lose-params-from-post-body-after-read-it-once/17129256#17129256)
