# Intro

## 스프링부트소개

### 강사소개

- N/A

### 준비사항

- SpringBoot 2.1.x
- Java Jdk8
- Spring 5.x
- Intellij, Eclipse, VSCode 
- 강의환경 : MacOS

## 스프링부트란?

### 스프링부트의 탄생

- SpringFramework의 설정이 너무 복합하고 어려웠기 때문에, 이를 해결하기 위해 만들어짐
- SpringBoot가 나오면서 초기 생산성이 획기적으로 좋아짐

### 스프링과 스프링부트의 차이점

- 스프링은 다양한 식재료이고, 스프링부트는 완성된 케이크라고 볼 수 있음
- 식재료를 다양하게 섞어서 자신만의 요리를 만들고 싶은 사람도 있을 것이고, 거의 다 만들어져 있는 음식을 데우기만 해서 먹고 싶은 사람도 있을 것

### 스프링부트의 특징

- Starter를 통한 어플리케이션의 간편하고 자동화된 빌드 및 설정을 제공함
- Embed 서버를 제공함으로써 복잡한 배포 설정들을 간편하게 제공함
- Production에서 사용할 수 있는 추가적인 기능을 제공함 (actuator)

### 웹설정 예제

#### Xml Configuration

```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/dispatcher-config.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
```

#### Xml + Java Configuration

```java
public class MyWebAppInitializer implements MyWebAppInitializer {
	@Override
	public void onStartup(ServletContext container) {
		XmlWebApplicationContext context = new XmlWebApplicationContext();
		context.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

		ServletRegistration.Dynamic dispatcher = container.addServlet("dispatcher", new DispatcherServlet(context));

		dispatcher.setLoadOnStartup(1);
		dispatcher.addMapping("/api");
	}
}
```

#### Only Java Configuration

```java
public class MyWebAppInitializer implements MyWebAppInitializer {
	@Override
	public void onStartup(ServletContext container) {
		AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
		context.setConfigLocation("com.fastcampus.myapp.config");

		container.addListener(new ContextLoadListener(context));

		ServletRegistration.Dynamic dispatcher = container.addServlet("dispatcher", new DispatcherServlet(context));

		dispatcher.setLoadOnStartup(1);
		dispatcher.addMapping("/api");
	}
}
```

#### SpringBoot Configuration

```
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
server.port=8081
server.context-path=/api
```

### Convention Over Configuration

- 설정보다는 관습 (줄여서 CoC, Coding by convention)
- 개발자가 설정해야 하는 것은 어플리케이션의 관례를 따르지 않는 점 뿐이다.

### 챕터요약

- Spring과 SpringBoot는 다른 것이다
- SpringBoot는 Java의 생산성 향상을 가져왔다
- 개발자가 설정해야 하는 것은 관례를 따르지 않는 점 뿐이다
