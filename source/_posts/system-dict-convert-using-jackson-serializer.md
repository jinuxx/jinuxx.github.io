---
title: 使用 Jackson 的 Serializer 进行系统字典翻译
tags: Java
date: 2021-10-12 16:06:32
---

对于部分实体类，字典值需要转换成中文，然后返回给前端。有几类解决方案：
1. 如果连表查询，侵入性比较大。
2. 使用 Aop，总体可行，但是代码量多，而且拆解嵌套对象，非常复杂。
3. 使用 Serializer，相对简单一些。

<!-- more -->
首先定义注解，用来标记需要转换的字段：
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside // 注意这个注解，声明之后下一行的注解才会生效
@JsonSerialize(using = DictSerializer.class)
public @interface Dict {

    /** 字典编码 */
    String value();
}
```
然后自定义序列化：
```java
@NoArgsConstructor // lombok 声明了一个无参构造器
public class DictSerializer extends JsonSerializer<Object> implements ContextualSerializer {

    private String code;

    public DictSerializer(String code) {
        this.code = code;
    }

    @Override
    public void serialize(Object value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        String field = gen.getOutputContext().getCurrentName();
        gen.writeObject(value);  // 这行如果不写会报错，为什么？
        // 通过 code 获取值
        String text = obtainText(code, value);
        gen.writeStringField(field + "_text", text);  // 添加自定义的中文字段
    }

    /**
     * 这个方法继承自ContextualSerializer，从这里可以拿到注解上的自定义字段，也就是字典编码
     */
    @Override
    public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property) {
        Dict annotation = property.getAnnotation(Dict.class);
        return new DictSerializer(annotation.value());
    }

    private String obtainText(String code, Object value) {
        DictService dictService = SystemBaseApi.getBeanByType(DictService.class); // 调用了一个获取 bean 的方法，
        return dictService.obtainText(code, value); // 这里会走缓存
    }
}
```
使用时，只要在需要的字段上添加 `@Dict(#{type_code})` 即可。