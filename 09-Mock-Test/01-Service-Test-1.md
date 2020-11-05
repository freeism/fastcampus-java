# Service Test #1

## 이론

* Mock 테스트의 장점
  * 테스트를 더 빠르게 실행할 수 있음
  * 테스트를 더 구체적이고 세밀하게 할 수 있음
* @ExtendWith
  * 테스트를 진행할 컨테이너를 별도로 지정해줌
  * Junit4에서 @RunWith를 대체하는 어노테이션
* MockitoExtension
  * Mockito를 사용할 수 있도록 처리해줌
* @InjectMocks
  * @Mock으로 지정된 객체들을 생성해서, 테스트의 주체가 되는 클래스에 주입(Autowired)까지 해줌
* @Mock
  * 실제 Bean이 아니라 가짜 객체(Mock)를 만들어서 실제 Bean을 대체함
* when...thenReturn
  * Mock의 어떤 메소드와 파라미터가 매핑되는 경우, 결과값에 대해서 지정해줄 수 있음

## 실전코드

```groovy
plugins {
    id 'org.springframework.boot' version '2.1.6.RELEASE'
    id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.fastcampus.javaallinone.project3'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

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

```java
@ExtendWith(MockitoExtension.class)
class PersonServiceTest {
    @InjectMocks
    private PersonService personService;
    @Mock
    private PersonRepository personRepository;

    @Test
    void getPeopleByName() {
        when(personRepository.findByName("martin"))
            .thenReturn(Lists.newArrayList(new Person("martin")));

        List<Person> result = personService.getPeopleByName("martin");

        assertThat(result.size()).isEqualTo(1);
        assertThat(result.get(0).getName()).isEqualTo("martin");
    }
}
```
