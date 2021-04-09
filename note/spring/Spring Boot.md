### Spring Boot Features
#### SpringApplication
**Customizing SpringApplication**  
If the SpringApplication defaults are not to your taste, you can instead create a local instance and customize it. 
For example, to turn off the banner, you could write:
```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```
It is also possible to configure the SpringApplication by using an application.properties file.  

**Application Availability**  
2.3新增，业务应用可以通过注入ApplicationAvailability来获取Liveness State和Readiness State，也可以订阅这两个actuator的变更事件：
```java
@Component
public class ReadinessStateExporter {

    @EventListener
    public void onStateChange(AvailabilityChangeEvent<ReadinessState> event) {
        switch (event.getState()) {
        case ACCEPTING_TRAFFIC:
            // create file /tmp/healthy
        break;
        case REFUSING_TRAFFIC:
            // remove file /tmp/healthy
        break;
        }
    }

}
```
业务应用可以通过Spring系统事件机制来修改Liveness State和Readiness State(此时/actuator/health/liveness和/actuator/health/readiness
的返回值都会发生变更):
```java
@Component
public class LocalCacheVerifier {

    private final ApplicationEventPublisher eventPublisher;

    public LocalCacheVerifier(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public void checkLocalCache() {
        try {
            //...
        }
        catch (CacheCompletelyBrokenException ex) {
            AvailabilityChangeEvent.publish(this.eventPublisher, ex, LivenessState.BROKEN);
        }
    }

}
```
**Accessing Application Arguments**
通过注入org.springframework.boot.ApplicationArguments bean获取参数。

**Using the ApplicationRunner or CommandLineRunner**
If you need to run some specific code once the SpringApplication has started, you can implement the ApplicationRunner or
CommandLineRunner interfaces.

#### Externalized Configuration
**Importing Additional Data**
可以通过spring.config.import在application.properties中引入其他配置，而且引入的配置优先级更高。

**Type-safe Configuration Properties**  
@ConfigurationProperties将配置文件中的配置属性映射到相应的Java Bean。需要配合@Component,@EnableConfigurationProperties或
@ConfigurationPropertiesScan使用：  
使用方式1：配合@Component使用  
![img.png](../../images/img.png)  
使用方式2：使用@EnableConfigurationProperties告知Spring Boot开启支持（也可用在带有@Configuration注解的类上）  
![img_1.png](../../images/img_1.png)  
使用方式3：配合@ConfigurationPropertiesScan使用（也可以加载启动类上）  
![img_2.png](../../images/img_2.png)