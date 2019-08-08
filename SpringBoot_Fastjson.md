#SpringBoot 整合 fastjson
## Springboot处理返回的参数为null、或者不返回
## 一、通过继承WebMvcConfigurerAdapter，重写configureMessageConverters方法实现

   ```
    @Configuration
    public class fastJsonConfig extends WebMvcConfigurerAdapter {
        @Autowired
        private LogCostInterceptor logCostInterceptor;
    
        /**
         * 使用阿里 fastjson 作为JSON MessageConverter
         * @param converters
         */
        @Override
        public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
            FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
            FastJsonConfig config = new FastJsonConfig();
            config.setSerializerFeatures(
                    //美化json输出，否则会作为整行输出
                    SerializerFeature.PrettyFormat,
                    // 保留map空的字段
                    SerializerFeature.WriteMapNullValue,
                    // 将String类型的null转成""
                    SerializerFeature.WriteNullStringAsEmpty,
                    // 将Number类型的null转成0
                    SerializerFeature.WriteNullNumberAsZero,
                    // 将List类型的null转成[]
                    SerializerFeature.WriteNullListAsEmpty,
                    // 将Boolean类型的null转成false
                    SerializerFeature.WriteNullBooleanAsFalse,
                    // 避免循环引用
                    SerializerFeature.DisableCircularReferenceDetect);
            converter.setFastJsonConfig(config);
            converter.setDefaultCharset(Charset.forName("UTF-8"));
            List<MediaType> mediaTypeList = new ArrayList<>();
            // 解决中文乱码问题，相当于在Controller上的@RequestMapping中加了个属性produces = "application/json"
            mediaTypeList.add(MediaType.APPLICATION_JSON);
            converter.setSupportedMediaTypes(mediaTypeList);
            converters.add(converter);
        }
    }
   ```
## 二、在Springboot启动类中
   ```
    @Bean
    public HttpMessageConverters fastJsonHttpMessageConverters() {
        FastJsonHttpMessageConverter4 fastConverter = new FastJsonHttpMessageConverter4();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat, SerializerFeature.IgnoreNonFieldGetter,
                SerializerFeature.WriteMapNullValue, SerializerFeature.WriteNullStringAsEmpty);
        fastConverter.setFastJsonConfig(fastJsonConfig);
        List supportedMediaTypes = new ArrayList();
        supportedMediaTypes.add(new MediaType("text", "json", Charset.forName("utf8")));
        supportedMediaTypes.add(new MediaType("application", "json", Charset.forName("utf8")));
        fastConverter.setSupportedMediaTypes(supportedMediaTypes);
        HttpMessageConverter<?> converter = fastConverter;
        return new HttpMessageConverters(converter);
    }
   ```
## 三、 在实体类中，添加： @JSONField注解，如果为false，接口中不会返回这个字段。
   ```
    @JSONField(serialize = false)
    private String password;
   ```
