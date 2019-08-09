#Springboot 默认的json处理方式 Jackson
## Springboot  处理返回参数的设置问题,接上篇SpringBoot 整合 fastjson
## 配置文件
   ```
    package com.summer.isnow.config;
    import com.fasterxml.jackson.core.JsonGenerator;
    import com.fasterxml.jackson.databind.JsonSerializer;
    import com.fasterxml.jackson.databind.ObjectMapper;
    import com.fasterxml.jackson.databind.SerializerProvider;
    import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
    
    import java.io.IOException;
    /**
     * @author liudongting
     * @date 2019/8/9 10:29
     */
    @Configuration
    public class JacksonConfig {
        @Bean
        @Primary
        @ConditionalOnMissingBean(ObjectMapper.class)
        public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
            ObjectMapper objectMapper = builder.createXmlMapper(false).build();
            objectMapper.getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object>() {
                @Override
                public void serialize(Object o, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                    jsonGenerator.writeString("");
                }
            });
            return objectMapper;
        }
    }
   ```
## 实体类
   ```
   	@JsonView(value = View.Base.class )
   	private Map<String,String> map;
   	@JsonView(value = View.Base.class )
   	private String []  ss;
   	@JsonView(value = View.Base.class )
   	private int [] intDemo;
   	@JsonView(value = View.Base.class )
   	private Integer b =null;
   	@JsonView(value = View.Base.class )
   	private boolean bbbb ;
   	@JsonView(value = View.Base.class )
   	private List<String> dd;
   ```
## 返回结果
   ![配置后，实体对应参数值得返回结果](https://img-blog.csdnimg.cn/20190809173621356.png)

## 结合上篇地址 
[SpringBoot 整合 fastjson](https://yq.aliyun.com/articles/713119?spm=a2c4e.11155435.0.0.79583312yq71FA)
