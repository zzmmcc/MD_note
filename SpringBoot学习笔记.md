---
title: SpringBoot学习笔记
date: 2019-06-29 15:11:05
tags: 笔记
---

#**SpringBoot笔记**
##微服务
    一种架构风格:一个应用应该是一组小型服务，可以通过http进行互通
    每一个功能元素最终都是一个可以独立替换和独立升级的软件单元
##常用注解 

```java
@SpringBootApplication      //标注这是一个主程序类，说明是一个springboot应用
@EnbaleAutoConfiguration    //开启自动配置功能
@SpringBootConfiguration    //标注这是一个SpringBoot的配置类
@RestController   //相当于@Controller+@ResponseBody
@ConfigurationProperties(prefix="")    //将全局配置文件中“prefix”的每一个值,映射到这个组件中 ,需要配置文件处理器,只有这个组件是容器的组件，才能使用容器的功能(需要交给spring容器管理@Component)
@RunWith(SpringRunner.class)+@SpringBootTest    //搭配使用，springboot的单元测试
@PropertySource(value={"classpath: xxx.properties"})        //加载指定的配置文件
@ImportResource(locations={"classpath: xxx.xml"})     //导入spring的配置文件，让配置文件中的内容生效 (类似以前的spring、springmvc的配置文件)
@Configuration      //声明这是一个配置类，搭配@Bean使用
@Bean       //类似于配置文件中的<bean></bean>标签，默认id为方法名
​```
```

##SpringBoot自动配置原理

1. SpringBoot启动是加载主配置类，开启了配置功能`@EnbaleAutoConfiguration`

   - `@EnbaleAutoConfiguration`的作用是利用`AutoConfigurationImportSelector`给容器导入一些组件:
   - 可以查看`selectImports()`方法的内容
   - `List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);`
   - `SpringFactoriesLoader.loadFactoryNames()`扫描所有jar
   - 包类路径下的META-INF/spring.factories，把扫描到的文件的内容包装成properties对象
   - 从properties中获取到`EnableAutoConfiguration.class`(类名)对应的值，然后把他们添加到容器中
   - *总结为:将类路径下META-INF/spring.factories里面配置所有的EnableAutoConfiguration的值加入到了容器中*

2. spring.factories
       ```properties
       # Auto Configure
       org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
       org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
       org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
       org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
       org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
       org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
       org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
       org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration
       /*每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置*/
       ```

3. 每一个自动配置类进行自动配置功能

4. 以`HttpEncodingAutoConfiguration`类为例，解释自动配置原理

   ```java
   @Configuration  //表示这是一个配置类，类似配置文件，也可以给容器添加组件
   @EnableConfigurationProperties({HttpProperties.class})  //启用指定类的ConfigurationProperties功能，将配置文件中对应的值和HttpEncodingProperties绑定，并把HttpEncodingProperties加入到ioc容器中
       //Spring底层有一个@Conditional注解，根据不停条件，如果满足指定的条件，整个配置类里面的配置才会生效
   @ConditionalOnWebApplication(type = Type.SERVLET)       //判断当前应用是否是web
   @ConditionalOnClass({CharacterEncodingFilter.class})        //判断当前项目有没有CharacterEncodingFilter类，SpringMVC进行解决乱码的过滤器
   @ConditionalOnProperty(prefix = "spring.http.encoding",value = {"enabled"},matchIfMissing = true)       //判断配置文件中是否存在spring.http.encoding.value的配置，如果不存在，判断也是成立的
   //即使配置文件中不配置此属性，也是默认生效的
   public class HttpEncodingAutoConfiguration {
       private final Encoding properties;  //已经和SpringBoot的配置文件映射了
       //只有一个有参构造器的情况下，参数的值会从容器中拿
        public HttpEncodingAutoConfiguration(HttpProperties properties) {
           this.properties = properties.getEncoding();
       }
       @Bean   //给容器添加一个组件，组件的某些值需要从properties中获取
       @ConditionalOnMissingBean
       public CharacterEncodingFilter characterEncodingFilter() {
           CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
           filter.setEncoding(this.properties.getCharset().name());
           filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
           filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
           return filter;
       }
   ```

      *根据当前不同条件判断，决定这个配置类是否生效，一旦生效，这个配置类就会给容器添加各种属性，组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是个配置文件绑定的*

5. 所有在配置文件能配置的熟悉都是在xxxProperties类中封装着的，配置文件能配置什么可以参照某个功能对应的这个属性类

   ```java
   @ConfigurationProperties(prefix = "spring.http")    //从配置文件获取指定的值和bean的属性进行绑定
   public class HttpProperties {
   ```

   ##SpringBoot的精髓
   1)、SpringBoot启动时会加载大量的自动配置类
   2)、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类
   3)、看自动配置类中到底配置了那些组件(如果我们需要的组件有，就不需要再来配置了)
   4)、给容器中自动配置类添加组件的时候，会从Properties雷总获取默写属性，我们可以在配置文件中去指定这些属性的值
   xxxxProperties:封装配置文件中的相关属性——>xxxxAutoConfiguration:自动配置类——>给容器添加组件
   ###**自动配置类必须在一定条件下才能生效(@Conditional及其衍生注解进行判断)**
   可以通过启用SpringBoot的debug=true属性，让控制台打印自动匹配值报告，知道哪些自动配置类生效

```java
============================
CONDITIONS EVALUATION REPORT
============================
Positive matches:   //已启用的
-----------------
   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer' (OnClassCondition)
      
Negative matches:   //未启用的，没有匹配成功的自动配置类
-----------------
ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
```

##SpringBoot与日志
    SpringBoot底层是使用slf4j+logback的方式记录进行日志记录
    SpringBoot也把其他的日志都替换成了slf4j(使用替换包
   **SpringBoot能自动适配所有日志，而且底层使用的是slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志包排除掉**

```xml
<dependency>
      <groupId>org.springframework.ws</groupId>
      <artifactId>spring-ws-core</artifactId>
      <version>3.0.7.RELEASE</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>
          <artifactId>commons-logging</artifactId>
          <groupId>commons-logging</groupId>
        </exclusion>
      </exclusions>
      <optional>true</optional>
    </dependency>
```

常用log配置

```yml
logging:
  level: {com.zz: trace}
    #不指定路径在当前项目或指定目录下生成log日志，需要指定文件名，path和file只能存在一个  
  path: G:/springboot.log    
    #在指定路径下创建spring.log作为默认日志文件,当前为项目所在盘的/spring/log文件夹下
  file:  /spring/log  
     #控制台输出格式  ：时间格式 [线程名] 左对齐 日志级别 全类名 字数 日志信息 换行  
    console: "%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n" 
    file: "%d{yyyy-MM-dd} ==== [%thread] == %-5level === %logger{50} ==== %msg%n"   
```

##SpringBoot的Web开发

**SpringBoot对静态资源的映射规则**    

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {
  //可以设置和静态资源有关的参数，缓存时间等
```

```java
**	WebMvcAuotConfiguration：
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }
                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                    //静态资源文件映射
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }
            }
        }
        //映射欢迎页面
         @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext) {
            return new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
        }
**
```

###使用webjars的方式引入jq、bootstrap等静态资源
    导入的资源jar包结构如下，springboot已经有了相关映射配置(详见上段代码)
    使用的时候只需要请求“webjars/jquery/3.1.1/jquery.js"即可成功
   ![](H:/MarkEditor/_image/2019-06-15-19-34-51.jpg)

###访问当前项目的任何资源都去静态资源文件夹去映射

```java
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：   
/**
*   以上所有目录的静态资源都能被springboot映射
*   / 表示当前项目的根路径
*   欢迎页所在位置应该为以上文件夹的index.html文件
*/
```

##Thymeleaf 
   SpringBoot内置的tomcat默认不支持jsp，推荐使用Thymeleaf
####**1、引入Thymeleaf(maven引入starter)**
####**2、  在页面上引入(引入有语法提示)`<html xmlns:th="http://www.thymeleaf.org">`**
####**3、 Thymeleaf的语法规则:**
    1)、th:text  改变当前元素的文本内容
        th:任意html属性来替换原来的属性的值
    2)、表达式
        ${...}:OGNL实现，获取对象的属性、调用方法、使用内置的基本对象、内置的工具对象
        *{...}:选择表达式，和${}功能上基本一致的，配合th:object使用 th:object=${user}; *{name},*{sex}
        #{...}:获取国际化内容
        @{...}:定义url
        ~{...}:片段引用

##**springboot的配置需要注意的地方**

###在使用webjars之后，在前端页面上引入方式为：

```
<script th:src="@{webjars/jquert/3.1.0/dist/jquery.js"></script>
<link rel="stylesheet" th:href="@{webjars/bootstrap/4.3.1/css/bootstrap.css}">
<script th:src="@{webjars/bootstrap/4.3.1/js/bootstrap.js}" type="text/javascript"></script>

```

##SpringBoot对SpringMVC的自动配置
 [官方文档](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-developing-web-applications.html)
###SpringBoot自动配置好了SpringMVC
以下是SpringBoot对SpringMVC的部分默认配置:*具体配置在org.springframework.boot.autoconfigure.web包中*

- Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
  *自动配置了ViewResolver(视图解析器:根据方法的返回值得到视图对象View，视图对象决定如何渲染_(转发，重定向))；ContentNegotiatingViewResolver : 组合所有的视图解析器，自动的将其组合进来*
- Support for serving static resources, including support for WebJars (covered later in this document)).
  *静态资源文件夹路径和webjars*
- Automatic registration of Converter, GenericConverter, and Formatter beans.
  *自动注册了Converter(转换器)，Formatter(格式化器，如日期格式化)*
- Support for HttpMessageConverters (covered later in this document).
  *HttpMessageConverters(消息转换器，SpringMVC用来转换Http请求和响应的，HttpMessageConverters是从容器中确定的)*
- Automatic registration of MessageCodesResolver (covered later in this document).*定义错误代码生成规则*
- Static index.html support.  *静态首页访问*
- Custom Favicon support (covered later in this document).  *favicon.ico图标*
- Automatic use of a ConfigurableWebBindingInitializer bean (covered later in this document).
  ####扩展SpringMVC
   编写配置类，使用@Configuration注解的类，是WebMvcConfigurer类型的；不能标注@EnableWebMvc

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/logout").setViewName("admin/success");
    }
}

```

原理:
    1)、WebMvcConfiguration是springboot的自动配置类
    2)、在做其他自动配置时会导入:@Import(EnableWebMvcConfiguration.class)

```java
 @Configuration
    public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {

```

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
    private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

    public DelegatingWebMvcConfiguration() {
    }

    @Autowired(required = false)    //从容器中获取所有的WebMvcConfigurer
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
        }
    }
```

  3)、容器中所有的WebMvcConfigurer都会一起起作用
  4)、我们自己写的配置类也会被调用

效果:SpringMVC的自动配置和扩展配置都会起作用
###全面接管SpringMVC(默认配置不会起作用，完全由自己配置SpringMVC)
@EnableWebMvc:

###SpringBoot定制错误页面或json数据
    1)、有模板引擎的情况下，在模板引擎文件夹下创建error文件夹，将对应的错误状态码.html放入其中(可以命名为4xx.html,5xx.html，优先寻找精确页面)
    2)、