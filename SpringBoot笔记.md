#**SSM笔记**
##AOP：
	使用注解需要在切面类上添加
	@Aspect			声明是一个切面类
	@Component		
	@EnableAspectJAutoProxy			开启自动代理
	使用配置文件需要引入org.aspectj.aspectjweaver
	<aop:config>		父级标签
		<aop:aspect id="" ref="引入切面类bean的id">
			<aop:before method="切面类中的某个方法名（只需要方法名，不要括号）"  pointcut="execution（* *..*.*（..））"/>
			<aop:around method="printEditLog" pointcut="execution(* *..*ServiceImpl.edit(..))"></aop:around>
			<!-- 
				使用around的时候 需要在增强方法传入一个ProceedingJoinPoint类型的对象，在该对象.proceed();前后各执行一个方法，这是环绕通知
				public void printEditLog(ProceedingJoinPoint pjp){
					printQuerytLog();
					try {
						pjp.proceed();
					} catch (Throwable throwable) {
						throwable.printStackTrace();
					}
					printEdit();
				}
			-->
		</aop:aspect>
	</aop:config>
##SpringMVC:
	其中关于日期的转换问题：
		1.前端传值到方法中进行参数绑定时，会出现日期格式为null，此时需要写一个日期转换类，继承Converter接口
				public class DateConverter implements Converter<String, Date> {}
			然后在springmvc.xml文件进行注册
				<bean class="org.springframework.context.support.ConversionServiceFactoryBean" id="converterDate">
					<property name="converters">
						<set>
							<bean class="com.zz.util.DateConverter"></bean>
						</set>
					</property>
				</bean>
			在mvc的注解驱动中绑定
				<mvc:annotation-driven conversion-service="converterDate"/>
		2.后端返回json类型的实体数据带有日期格式，前端接受的为此时间到1970-01-01的毫秒数，需要在实体类继承JsonSerializer<Date> 
			重写方法public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
						SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
						String s = sdf.format(date);
						jsonGenerator.writeString(s);
					}
			在日期类型的get方法上@JsonSerialize(using = 实体类.class)
	springMVC实现文件上传：
		首先需要在springmvc.xml文件配置
			<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
				<property name="maxUploadSize" value="104857600"></property>
			</bean>
		然后在tomcat服务器的配置文件中配置（tomcat X.X/conf/web.xml的servlet标签中）
			<init-param>
				<param-name>readonly</param-name>
				<param-value>false</param-value>
			</init-param>
		1.同服务器上传：
			public String uploadTest(HttpServletRequest request, MultipartFile upload) throws IOException {
				String path = request.getSession().getServletContext().getRealPath("/upload").toString();
				File file = new File(path);
				if (!file.exists()){
					file.mkdirs();
				}
				String filename = upload.getOriginalFilename();
				filename = UUID.randomUUID().toString()+"-"+filename;
				upload.transferTo(new File(file,filename));
				return "index";
			}
		2.跨服务器上传：
			public String uploadOther(MultipartFile upload) throws IOException {
				String path = "http://localhost:8081/upload/";
				File file = new File("/upload");
				if (!file.exists()){
					file.mkdirs();
				}
				String filename = upload.getOriginalFilename();
				Client client = Client.create();
				WebResource resource = client.resource(path + filename);
				resource.put(upload.getBytes());
				return "index";
			}
##SSM整合：
	引入jar包
		Spring(context,aop,tx,aspectjweaver,web,webmvc,jdbc,test)
		MyBatis(mybatis,mybatis-spring)
		其他(junit,mysql,druid,jackson-core/databind/annotations,log4j,slf4j-log4j12)
	配置web.xml
		配置监听器(org.springframework.web.context.ContextLoaderListener)
		通过上下文加载applicationContext.xml
		配置前端控制器
			class:org.springframework.web.servlet.DispatcherServlet
			通过指定init-para来加载spring-mvc.xml
		配置字符编码过滤器......
	编写实体类、mapper接口、mapper.xml
		配置applicationContext.xml
		扫包（必要）
		配置数据源：
			<bean id="datasource" class="com.alibaba.druid.pool.DruidDataSource"> 注入属性</bean>
		整合mybatis
			将mybatis的sqlSessionFactory交给spring管理(class="org.mybatis.spring.SqlSessionFactoryBean")
				注入属性：数据源,mapper.xml映射文件的路径，配置实体别名
			配置mapper接口的包所在位置，让spring容器管理(class="org.mabatis.spring.mapper.MapperScannerConfigurer")
				注入属性：mapper接口所在位置，sqlSessionFactoryBeanName		!!!!!!注入sqlSessionFactory时不要使用ref，要用value
		配置事务管理器
			class="org.springframework.jdbc.datasource.DataSourceTransactionManager"	注入数据源
		开启对事务注解的支持：<tx:annotation-driven transaction-manager="事务管理器的id" />
	配置spring-mvc.xml
		扫包（必要）
		开启对mvc注解的支持
		配置视图解析器：class="org.springframework.web.servlet.view.InternalResourceViewResolver" 
			配置前缀，后缀
		配置静态资源
			配置全局异常处理类时可以在spring的配置文件也可以在springmvc的配置文件
		
#***SpringBoot笔记***
##微服务
    一种架构风格:一个应用应该是一组小型服务，可以通过http进行互通
    每一个功能元素最终都是一个可以独立替换和独立升级的软件单元
#**SpringBoot+Thymeleaf** 
##**springboot的配置需要注意的地方**
### 在使用webjars之后，在前端页面上引入方式为：
    <script th:src="@{webjars/jquert/3.1.0/dist/jquery.js"></script>
    <link rel="stylesheet" th:href="@{webjars/bootstrap/4.3.1/css/bootstrap.css}">
    <script th:src="@{webjars/bootstrap/4.3.1/js/bootstrap.js}" type="text/javascript"></script>
##**注解**
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
```




