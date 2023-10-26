---
layout: default
title: SpringBoot Static
parent: SpringBoot
nav_order: 130
---


## SpringBoot 静态资源配置

在 SpringBoot 的特征中之一是 SpringBoot 的自动化配置。 SpringBoot 已经默认将大部分场景配置都做好了，静态资源部分在其中。

### 1、静态资源管理

静态资源的配置在 SringMVC 的自动化配置类  `org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration`中进行管理的。真正实现的是在其内部静态类 `WebMvcAutoConfigurationAdapter` 中进行的一系列的自动化配置。

```java
@Configuration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
     // some code 
    
	@Configuration
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter
			implements WebMvcConfigurer, ResourceLoaderAware {

		private static final Log logger = LogFactory.getLog(WebMvcConfigurer.class);

		private final ResourceProperties resourceProperties;
        
         // some code 
	}
}
```

 其中 `ResourceProperties` 就是静态静态资源配置的类，可以设置和静态资源有关的参数，资源路径、是否缓存、缓存周期（秒）、资源处理链、资源版本是否固定一系列内容。

**常见的以 xxxxProperties 的配置类是用来封装配置文件的内容的 Bean 。**

在 `WebMvcAutoConfiguration` 的内部静态类 `WebMvcAutoConfigurationAdapter`中进行一些常见的静态资源处理：

```java
 		// 静态资源异步支持
		@Override
		public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
			if (this.beanFactory.containsBean(
					TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME)) {
				Object taskExecutor = this.beanFactory.getBean(
						TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME);
				if (taskExecutor instanceof AsyncTaskExecutor) {
					configurer.setTaskExecutor(((AsyncTaskExecutor) taskExecutor));
				}
			}
			Duration timeout = this.mvcProperties.getAsync().getRequestTimeout();
			if (timeout != null) {
				configurer.setDefaultTimeout(timeout.toMillis());
			}
		}
		// 国际化
		@Bean
		@ConditionalOnMissingBean
		@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
		public LocaleResolver localeResolver() {
			if (this.mvcProperties
					.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
				return new FixedLocaleResolver(this.mvcProperties.getLocale());
			}
			AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
			localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
			return localeResolver;
		}
		// 静态资源映射相关
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache()
					.getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry
						.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod))
						.setCacheControl(cacheControl));
			}
            	// 静态资源文件夹映射处理
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(getResourceLocations(
										this.resourceProperties.getStaticLocations()))
								.setCachePeriod(getSeconds(cachePeriod))
								.setCacheControl(cacheControl));
			}
		}
		// 欢迎页 静态资源文件夹下的所有index.html页面；被"/**"映射
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(
				ApplicationContext applicationContext) {
			return new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext),
					applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
		}
		//  favicon 图标
		@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration implements ResourceLoaderAware {

			private final ResourceProperties resourceProperties;

			private ResourceLoader resourceLoader;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Override
			public void setResourceLoader(ResourceLoader resourceLoader) {
				this.resourceLoader = resourceLoader;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
                // 所有的 **/favicon.ico  都是在静态资源文件下找
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler.setLocations(resolveFaviconLocations());
				return requestHandler;
			}

			private List<Resource> resolveFaviconLocations() {
				String[] staticLocations = getResourceLocations(
						this.resourceProperties.getStaticLocations());
				List<Resource> locations = new ArrayList<>(staticLocations.length + 1);
				Arrays.stream(staticLocations).map(this.resourceLoader::getResource)
						.forEach(locations::add);
				locations.add(new ClassPathResource("/"));
				return Collections.unmodifiableList(locations);
			}

		}
```

还有一些视图解析、路径匹配等等相关处理。



### 2、静态资源默认配置

在 ResourceProperties  类中可以找点默认的静态资源路径

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
    
    // some code 
}
```

默认路径有：

```txt
"classpath:/META-INF/resources/"
"classpath:/resources/"
"classpath:/static/"
"classpath:/public/"
```

静态资源默认可以放这4个地方。



### 3、静态资源自定义配置

在 YAML 语法学习 篇章中学习了 配置文件与JavaBean 的绑定，其实 **`@ConfigurationProperties`** 就是其中方式之一，并且这个类没有使用**`@PropertySource`** 注解用来指定文件，说明如果要自定义配置以 `spring.resources` 前缀的静态资源相关配置必须放在全局配置`application.properties` 或 `application.yml` 里面。如果写在另外的配置文件将无法识别。当然也支持Java方式设置自定义配置。

比如在 `application.properties`中：

```properties
spring.resources.static-locations=/**
```
其中 `static-locations` 就是 `ResourceProperties` 中的 `staticLocations` 属性，它是一个数组，可以配置多个静态资源路径，以逗号 `,` 分隔。

`static-locations` 也可以写成 `static_locations` 。

如果 SpringBoot 版本较高，静态资源可能略有变化：

```properties
spring.mvc.static-path-pattern=/resources/**
```

可以到 `ResourceProperties` 类中查看一下前缀和变量即可。

### 4、静态资源引入

对于静态资源引入可以两种：

- （1）直接引入外部库 `webjars` 方式。
- （2）引入自定义静态资源，比如项目自定义的`js` 、`css` 、`images`等

（1）`webjars` 方式

`webjars` 是以jar包的方式引入静态资源，可以在 webjars 网站 http://www.webjars.org/ 查询引入自己需要的。

如引入 jquery :

```xml
<!--引入jquery-webjar-->
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>jquery</artifactId>
			<version>3.3.1</version>
		</dependency>
```

在访问的时候只需要写 webjars 下面资源的名称即可，如 `localhost:8080/webjars/jquery/3.3.1/jquery.js` 

所有 /webjars/** ，都会在 classpath:/META-INF/resources/webjars/ 找对应的资源。

（2）自定义方式

`"/**"` 访问当前项目的任何资源，都去（静态资源的文件夹）找映射自定义方式前面提到了默认路径：

```txt
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
```

### 5、其他

如果遇到其他问题后续补充

 

​	

