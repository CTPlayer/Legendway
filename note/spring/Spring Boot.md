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