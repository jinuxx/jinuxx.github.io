---
title: Spring-boot下接口统一返回
tags:
  - Java
  - Spring
date: 2023-02-10 14:53:11
---


Api 接口返回值需要统一分装在
```json
{
  "code": 0,
  "msg": "",
  "data": <DATA>
}
```
中（需要吗？），通过 spring boot 的 `ResponseBodyAdvice` 实现:

<!-- more -->
结果包装类：
```java
@Getter
@Setter
@NoArgsConstructor
public class ResultWrapper<T> implements Serializable {

    private static final long serialVersionUID = -1225323124494233417L;

    /** 响应码 */
    private Integer code;
    /** 返回错误信息 */
    private String msg = "";
    /** 返回数据 */
    private T data;

    /**
     * 成功不带返回数据
     * @author W
     * @create_at 2019/3/29 10:56
     */
    public static <T> ResultWrapper<T> success() {
        ResultWrapper<T> resultWrapper = new ResultWrapper<>();
        resultWrapper.setCode(0);
        return resultWrapper;
    }

    /**
     * 成功带返回数据
     * @author W
     * @create_at 2019/3/29 10:56
     */
    public static <T> ResultWrapper<T> success(T data) {
        ResultWrapper<T> resultWrapper = new ResultWrapper<>();
        resultWrapper.setCode(0);
        resultWrapper.setData(data);
        return resultWrapper;
    }

    /**
     * 失败，入参为错误码与消息
     * @author XuJin
     * @create_at 2019/5/9 15:56
     */
    public static <T> ResultWrapper<T> error(Integer code, String msg) {
        ResultWrapper<T> resultWrapper = new ResultWrapper<>();
        resultWrapper.setCode(code);
        resultWrapper.setMsg(msg);
        return resultWrapper;
    }

    /**
     * 失败，入参为错误码与消息
     * @author XuJin
     * @create_at 2019/5/9 15:56
     */
    public static <T> ResultWrapper<T> error(String msg) {
        return error(500, msg);
    }

    public ResultWrapper<T> setMsg(String msg) {
        this.msg = msg;
        return this;
    }
}
```

`ResponseBodyAdvice`实现类，在这里包装
```java
@Slf4j
@RequiredArgsConstructor
@RestControllerAdvice(annotations = RestController.class, basePackages = {"com.example"})
public class ResponseWrapperHandler implements ResponseBodyAdvice<Object> {

    private final ObjectMapper objectMapper;

    @Override
    public boolean supports(MethodParameter methodParameter, Class<? extends HttpMessageConverter<?>> converterType) {
        return !List.of(
                ResultWrapper.class,
                ResponseEntity.class,
                File.class
        ).contains(methodParameter.getParameterType());
    }

    @Override
    public Object beforeBodyWrite(Object body,
                                  MethodParameter methodParameter,
                                  MediaType selectedContentType,
                                  Class<? extends HttpMessageConverter<?>> selectedConverterType,
                                  ServerHttpRequest request,
                                  ServerHttpResponse response) {
        ResultWrapper<Object> wrappedResult = ResultWrapper.success(body);
        // 当接口返回类型是 String，而且
        if (String.class == methodParameter.getParameterType()) {
            try {
                return objectMapper.writeValueAsString(wrappedResult);
            } catch (JsonProcessingException e) {
                throw new BizException(ResponseEnum.INTERNAL_SERVER_ERROR);
            }
        }
        return wrappedResult;
    }
}
```
这样在 `com.example` 下所有的接口返回都会被包装。

如果有的接口不需要包装而是直接返回，将接口的返回值通过 `ResponseEntity` 包装