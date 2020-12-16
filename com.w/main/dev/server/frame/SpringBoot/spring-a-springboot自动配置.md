# SpringBoot

## SpringApplication.run分析

分析该方法主要分两部分:

- 一是SpringApplication的实例化，
- 二是run方法的执行；

## 自动装配

### SpringBootApplication

注解源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

- SpringBootConfig---Spring配置类

    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Configuration
    public @interface SpringBootConfiguration {
    	@AliasFor(annotation = Configuration.class)
    	boolean proxyBeanMethods() default true;
    }
    ```

    - Configuration---Spring组件类

        ```java
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Component
        public @interface Configuration {
        	@AliasFor(annotation = Component.class)
        	String value() default "";
        	boolean proxyBeanMethods() default true;
        }
        ```

- EnableAutoConfiguration---开启自动配置

    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @AutoConfigurationPackage//---扫描主配置类，及其所在包
    @Import(AutoConfigurationImportSelector.class)
    public @interface EnableAutoConfiguration {
    	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    	Class<?>[] exclude() default {};
    	String[] excludeName() default {};
    }
    ```

    - @AutoConfigurationPackage---配置

        ```java
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @Import(AutoConfigurationPackages.Registrar.class)//为Spring导入组件，Registrar.class
        public @interface AutoConfigurationPackage {
        	String[] basePackages() default {};
        	Class<?>[] basePackageClasses() default {};
        }
        ```
        - Register.class---获取包路径

            ```java
            static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
            		@Override
            		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            			register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
                        //使用断点表达式运行
                        //---new AutoConfigurationPackages.PackageImports(metadata)
                        //---结果获取到了：包名
                        //此处的metadata---debug显示实质上是主配置类
            		}
            		@Override
            		public Set<Object> determineImports(AnnotationMetadata metadata) {
            			return Collections.singleton(new PackageImports(metadata));
            		}
            	}
            ```

    - @Import(AutoConfigurationImportSelector.class)

        ```java
        @Override
        public String[] selectImports(AnnotationMetadata annotationMetadata) {
            if (!isEnabled(annotationMetadata)) {
                return NO_IMPORTS;
            }
            AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);//调用方法
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
        //加载为容器加载组件
        protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
            if (!isEnabled(annotationMetadata)) {
                return EMPTY_ENTRY;
            }
            AnnotationAttributes attributes = getAttributes(annotationMetadata);
            List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);//获取配置类
            configurations = removeDuplicates(configurations);
            Set<String> exclusions = getExclusions(annotationMetadata, attributes);
            checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = getConfigurationClassFilter().filter(configurations);
            fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationEntry(configurations, exclusions);
        }
        //获取配置类
        protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
            List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
            Assert.notEmpty(configurations, 
                            "No auto configuration classes found in META-INF/spring.factories. If you "
                            + "are using a custom packaging, make sure that file is correct.");
            return configurations;
        }
        //执行获取配置的方法
        public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
            String factoryTypeName = factoryType.getName();
            return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
            //此处的loadSpringFactories方法中加载了配置文件内容
        }
        //加载配置文件
        private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        		MultiValueMap<String, String> result = cache.get(classLoader);
        		if (result != null) {
        			return result;
        		}
        		try {
        			Enumeration<URL> urls = (classLoader != null ?
        					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
        					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        			result = new LinkedMultiValueMap<>();
        			while (urls.hasMoreElements()) {
        				URL url = urls.nextElement();
        				UrlResource resource = new UrlResource(url);
        				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
        				for (Map.Entry<?, ?> entry : properties.entrySet()) {
        					String factoryTypeName = ((String) entry.getKey()).trim();
        					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
        						result.add(factoryTypeName, factoryImplementationName.trim());
        					}
        				}
        			}
        			cache.put(classLoader, result);
        			return result;
        		}
        		catch (IOException ex) {
        			throw new IllegalArgumentException("Unable to load factories from location [" +
        					FACTORIES_RESOURCE_LOCATION + "]", ex);
        		}
        	}
        ```
        
自动配置SpringBoot从**META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效**
        
该文件部分内容如下，可以看得到配置了很多的配置类的路径
        
```properties
        # AutoConfigureCache auto-configuration imports
        org.springframework.boot.test.autoconfigure.core.AutoConfigureCache=\
        org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration
        
        # AutoConfigureDataJdbc auto-configuration imports
        org.springframework.boot.test.autoconfigure.data.jdbc.AutoConfigureDataJdbc=\
        org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
        org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
        org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
        org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
        org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
        ...
        ```

## 总结

SpringBoot中通过配置@SpringBootApplication注解标记启动类，实质上是在主类上预先配置了很多的注解，例如自动导入等等，当Spring启动时会预先扫描配置的包，再加载在spring.factories中配置的类路径，通过这些预先配置好的类路径下的类实现对类的自动配置