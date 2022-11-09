---
title: Spring Boot 全局错误处理
date: 2021-10-14 14:49:49
tags: Java
---
我们一般用 `@[Rest]ControllerAdvice` 来做 `Controller` 的全局错误处理，比如在项目中：
```java
/**
 * 全局异常处理，需要自定义异常处理的
 * @author W
 * @create_at 2019/3/29 14:30
 */
@Slf4j
@RestControllerAdvice
public class ExceptionAdviceHandler {

    /**
     * 不支持的http方法  405
     * @author W
     * @create_at 2019/3/29 11:36
     */
    @ExceptionHandler(value = HttpRequestMethodNotSupportedException.class)
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)
    public ResultWrapper<?> methodNotAllowed() {
        return ResultWrapper.error(ResponseEnum.ERROR_REQUEST_METHOD);
    }

    @ExceptionHandler(NoHandlerFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResultWrapper<?> handleError404() {
        return ResultWrapper.error(ResponseEnum.NOT_FOUND);
    }

    // ...
}
```
<!-- more -->
但是有时候，在 `Filter` 或者 `Interceptor` 中，也会有错误抛出，这些错误就不能被捕获了。这时候可以用自定义的 `Filter` 去 `try-catch`

```java
// 在使用 SaToke 时，验证是否登录是通过 Interceptor 进行的，因此在这里捕获未登录的错误
@Component
@WebFilter(urlPatterns = "/**")
@Order(Ordered.HIGHEST_PRECEDENCE)
public class NotLoginFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        try {
            filterChain.doFilter(request, response);
        } catch (NestedServletException e) {
            if (e.getCause() instanceof NotLoginException) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
                response.getWriter().write(JsonUtils.objectToJson(ResultWrapper.error(ResponseEnum.INVALID_TOKEN)));
            } else {
                throw e;
            }
        }
    }
}
```

---
对于 404 错误的，默认是系统返回结果：
```json
{
    "timestamp": "2021-10-14 15:01:50",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/unknown"
}
```
现在想要统一返回值，需要配置 `application.properties`。~~并且不使用 `@EnableWebMvc`([stackoverflow](https://stackoverflow.com/questions/30917782/spring-boot-404-error-custom-error-response-rest?answertab=active#tab-top))，与 `ResponseBodyAdvice` 配置有冲突？TODO~~:

```properties
spring.mvc.throw-exception-if-no-handler-found=true
server.error.whitelabel.enabled=false
spring.resources.add-mappings=false
```

并添加 `ExceptionHandler`
```java
    @ExceptionHandler(NoHandlerFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResultWrapper<?> handleError404() {
        return ResultWrapper.error(ResponseEnum.NOT_FOUND);
    }
```