# 스프링부트 시작

## 스프링부트 시작하기

### 스프링부트 프로젝트를 생성하기

- https://start.spring.io 에서 생성하기
- IntelliJ > new Project > Spring Initializer 에서 생성하기

## HelloWorldController 생성하기

### Controller 생성

```java
@RestController
public class HelloWorldController {
	@GetMapping(value = "/api/helloWorld")
	public String helloWorld() {
		return "HelloWorld";
	}
}
```

### Embed WAS 서버실행

```bash
./gradlew bootRun
```

### 브라우저에서 테스트

```
http://localhost:8080/api/helloWorld
```

### http 파일을 통한 테스트

```
GET http://localhost:8080/api/helloWorld
```

## MockMvc 테스트 만들기

### gradle 의존성 추가

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'com.h2database:h2'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.junit.platform:junit-platform-launcher:1.5.0'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.5.0'
    testImplementation 'org.junit.jupiter:junit-jupiter-engine:5.5.0'
    testImplementation 'org.mockito:mockito-junit-jupiter'
}
```

### ControllerTest 생성

```java
@SpringBootTest
class HelloWorldControllerTest {
	@Test
	void helloWorld() {
		System.out.println("test");
	}
}
```

### HelloWorldController 테스트

```java 
@SpringBootTest
class HelloWorldControllerTest {
	@Autowired
	private HelloWorldController HelloWorldController;

	@Test
	void helloWorld() {
		assertThat(HelloWorldController.HelloWorldController()).isEqualTo("HelloWorld");
	}
}
```

### MockMvc 테스트

```java
@SpringBootTest
class HelloWorldControllerTest {
    @Autowired
	private HelloWorldController HelloWorldController;

    private MockMvc mockMvc;

    @Test
	void helloWorld() {
		assertThat(HelloWorldController.HelloWorldController()).isEqualTo("HelloWorld");
	}

    @Test
    void mockMvcTest() throws Exception {
    	mockMvc = MockMvcBuilders.standaloneSetup(HelloWorldController).build();

        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/helloWorld"))
        	.andDo(MockMvcResultHandlers.print())
            .andExpect(status().isOk())
            .andExpect(MockMvcResultMatchers.content().string("HelloWorld"));
    }
}
```