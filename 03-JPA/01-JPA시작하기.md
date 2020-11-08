# JPA 시작하기


## 이론

* JPA사용을 위한 의존성 추가하기
  * spring-boot-starter-data-jpa : JPA사용을 위한 스프링부트 스타터
  * h2 : 간단하게 사용할 수 있는 초경량 메모리 DB
* JPA의 특징
  * DB의 종류에 상관없이 JPA문법에 따라서 로직을 생성할 수 있음
  * DB를 변경하더라도 JPA에서 해당 DB에 적합한 쿼리를 자동으로 변경생성해줌
* 주요 Annotation의 역할
  * @Entity : 해당 클래스가 Domain Entity로 사용할 것이라는 표식
  * @Id : 해당 컬럼이 Domain Entity의 Pk임을 명시함
  * @GeneratedValue : 해당 컬럼의 값은 자동으로 생성되는 값임을 명시함
    * GenerationType : TABLE, SEQUENCE, IDENTITY, AUTO (default)
    
## 실전코드

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
}
```

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", age=" + age +
            '}';
    }
}
```

```java
public interface PersonRepository extends JpaRepository<Person, Long> {
}
```

```java
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void crud() {
        Person person = new Person();
        person.setName("martin");
        person.setAge(10);

        personRepository.save(person);

//        System.out.println(personRepository.findAll());

        List<Person> people = personRepository.findAll();

        assertThat(people.size()).isEqualTo(1);
        assertThat(people.get(0).getName()).isEqualTo("martin");
        assertThat(people.get(0).getAge()).isEqualTo(10);
    }
}
```

```yaml
spring:
  jpa:
    show-sql: true
```
