### The IoC Container
#### Bean Scopes
当一个scope为prototype的bean注入到singleton的bean中，由于singleton的bean只会实例化一次，所以导致作为
依赖的prototype的bean并不能拥有预期的生命周期。  
解决方式： 
* @Lookup
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```
* scope proxy
  @Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE,proxyMode = ScopedProxyMode.TARGET_CLASS)
  
#### Customizing the Nature of Bean
![img.png](../../images/20210507-1.jpg)
##### Lifecycle Callbacks
* Initialization Callbacks(优先级从高到低)  
  使用@PostConstruct注解；  
  实现org.springframework.beans.factory.InitializingBean接口，调用afterPropertiesSet()方法；  
  In the case of XML-based configuration metadata, you can use the init-method attribute to specify the name of the 
  method that has a void no-argument signature.  
* Destruction Callbacks(优先级从高到低)  
  使用@PreDestroy注解；  
  实现org.springframework.beans.factory.DisposableBean，调用destroy()方法；  
  With XML-based configuration metadata, you can use the destroy-method attribute on the <bean/>
  
##### 容器启动或停止回调
* Lifecycle接口
```java
public interface Lifecycle {
    // 当容器启动时调用
    void start();
    // 当容器停止时调用
    void stop();
    // 当前组件的运行状态
    boolean isRunning();
}
```
需要手动执行上下文start(),stop()方法，才能触发生命周期方法执行。  
* SmartLifecycle接口
  它本身除了继承了Lifecycle接口还继承了一个Phased接口，其接口定义如下：  
```java
public interface Phased { 
    /**    
     * Return the phase value of this object.    
     */
    int getPhase();
}
```
 通过上面接口定义的方法，我们可以指定不同Bean方法回调方法执行的优先级。  
```java
public interface SmartLifecycle extends Lifecycle, Phased {

    int DEFAULT_PHASE = Integer.MAX_VALUE;

    // 不需要显示的调用容器的start方法及stop方法也可以执行Bean的start方法跟stop方法
    default boolean isAutoStartup() {
        return true;
    }

    // 容器停止时调用的方法
    default void stop(Runnable callback) {
        stop();
        callback.run();
    }

    // 优先级，默认最低
    @Override
    default int getPhase() {
        return DEFAULT_PHASE;
    }

}
```
当我们启动容器时，如果有Bean实现了SmartLifecycle接口，其getPhase()方法返回的值越小，那么对于的start方法执行的时间就会越早，stop方法执行的时机就会越晚。
因此，一个实现SmartLifecycle的对象，它的getPhase()方法返回Integer.MIN_VALUE将是第一个执行start方法的Bean和最后一个执行Stop方法的Bean。  

##### ApplicationContextAware and BeanNameAware
提供获取ApplicationContext和BeanName的方式；  
The callback is invoked after population of normal bean properties but before an initialization callback such as InitializingBean,
afterPropertiesSet, or a custom init-method.  

#### Container Extension Points
##### Customizing Beans by Using a BeanPostProcessor（可以用来处理自定义注解，例如Autowired标签的解析逻辑主要在AutowiredAnnotationBeanPostProcessor类中）
```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```
当一个BeanPostProcessor的实现类注册到Spring IOC容器后，对于该Spring IOC容器所创建的每个bean实例在初始化方法（如afterPropertiesSet和任意已声明的init方法）
调用前，将会调用BeanPostProcessor中的postProcessBeforeInitialization方法，而在bean实例初始化方法调用完成后，
则会调用BeanPostProcessor中的postProcessAfterInitialization方法。  

spring容器通过BeanPostProcessor给了我们一个机会对Spring管理的bean进行再加工。

##### Customizing Configuration Metadata with a BeanFactoryPostProcessor
BeanFactoryPostProcessor接口可以在bean未被实例化之前获取bean的定义即配置元数据，然后根据需要进行更改。  

##### Customizing Instantiation Logic with a FactoryBean

Spring中有两种类型的Bean,一种是普通Bean,另一种是工厂Bean,即FactoryBean。Spring FactoryBean是创建复杂的bean,一般的bean直接用xml配置即可,
如果一个bean的创建过程中涉及到很多其他的bean和复杂的逻辑,用xml配置比较困难,这时可以考虑用FactoryBean。  

用法：https://juejin.cn/post/6844903954615107597

#### Classpath Scanning and Managed Components
##### Defining Bean Metadata within Components
如果使用 @Configuration 注解修饰的类，并且该注解中的 proxyBeanMethods 属性的值为 true，则会为这个 bean 创建一个代理类，该代理类会拦截所有被 @Bean
修饰的方法，在拦截的方法逻辑中，会从容器中返回所需要的单例对象。  
如果使用 @Component 注解修饰的类，则不会为这个 bean 创建一个代理类。 那么我们就会直接执行用户的方法，所以每次都会返回一个新的对象。  
如果将 @Configuration 注解中的 proxyBeanMethods 属性的值设置为 false，那么它的行为是否就会跟 @Component 注解一样。  

只要是@Component注解修饰的类里面都可以定义@Bean，并且都可以注册到Spring容器里面。其实不仅仅是@Component，只要是被@Component修饰的注解同样也可以定义@Bean，
比如：@Repository、@Service、@Controller @Configuration，甚至是接口的default方法上的@Bean也可以被扫描加入到Spring容器里面。  

##### Generating an Index of Candidate Components
在SpringFramework5.0引入了一个注解@Indexed ，它可以为Spring的模式注解添加索引，以提升应用启动性能。  

在应用中有大量使用@ComponentScan扫描的package包含的类越多的时候，Spring模式注解解析耗时就越长。  
在项目中使用的时候需要导入一个spring-context-indexer jar包，使用maven方式，引入jar配置如下：  
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.1.12.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```
然后在代码中，对于使用了模式注解的类上加上@Indexed注解即可。如下：  
```java
@Indexed
@Controller
public class HelloController {

}
```

在项目中使用了@Indexed之后，编译打包的时候会在项目中自动生成META-INT/spring.components文件。当Spring应用上下文执行ComponentScan扫描时，
META-INT/spring.components将会被CandidateComponentsIndexLoader 读取并加载，转换为CandidateComponentsIndex对象，
这样的话@ComponentScan不在扫描指定的package，而是读取CandidateComponentsIndex对象，从而达到提升性能的目的。  