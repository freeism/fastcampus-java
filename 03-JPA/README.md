# JPA

## JPA시작하기

### JPA사용을 위한 의존성 추가하기

* spring-boot-starter-data-jpa : JPA사용을 위한 스프링부트 스타터
* h2 : 간단하게 사용할 수 있는 초경량 메모리 DB

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

#### JPA의 특징

* DB의 종류에 상관없이 JPA문법에 따라서 로직을 생성할 수 있음
* DB를 변경하더라도 JPA에서 해당 DB에 적합한 쿼리를 자동으로 변경생성해줌

### Entity 생성하기

#### 주요 Annotation의 역할
* @Entity : 해당 클래스가 Domain Entity로 사용할 것이라는 표식
* @Id : 해당 컬럼이 Domain Entity의 Pk임을 명시함
* @GeneratedValue : 해당 컬럼의 값은 자동으로 생성되는 값임을 명시함
** GenerationType : TABLE, SEQUENCE, INDENTITY, AUTO (default)

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;
}
```

### Repository 생성하기

```java
public interface PersonRepository extends JpaRepository<Person, Long> {
}
```

### JPA 동작 확인하기 (테스트)

```java
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void crud() {
    	Person person = new Person();

    	personRepository.save(person);

    	System.out.println(personRepository.findAll());
    }
}
```

### 테스트의 보완

#### Person 객체의 해시값이 출력됨 -> 알 수 있는 데이터로 출력

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    @Override
    public String toString() {
    	return "Person{" +
    		"id=" + id +
    		", name='" + name + "\'"
    		", age=" + age +
    		"}";
    }
}
```

#### Person 객체의 field에 값을 셋팅할 수 있는 방법

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
    		", name='" + name + "\'"
    		", age=" + age +
    		"}";
    }
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

    	System.out.println(personRepository.findAll());
    }
}
```

#### 자동화된 테스트

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

//    	System.out.println(personRepository.findAll());
    	List<Person> people = personRepository.findAll();

    	assertThat(people.size()).isEqualTo(1);
    	assertThat(people.get(0).getName()).isEqualTo("martin");
    	assertThat(people.get(0).getAge()).isEqualTo(10);
    }
}
```

### JPA가 생성한 DB쿼리 확인

```yml
spring:
	jpa:
		show-sql: true
```

## Lombok (1)

### Lombok 의존성 추가 (사전에 추가함)

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

### Lombok @Getter, @Setter의 필요성 확인

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

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

    public String getHobby() {
    	return hobby;
    }

    public void setHobby(String hobby) {
    	this.hobby = hobby;
    }

    public String getBloodType() {
    	return bloodType;
    }

    public void setBloodType(String bloodType) {
    	this.bloodType = bloodType;
    }

    public String getAddress() {
    	return address;
    }

    public void setAddress(String address) {
    	this.address = address;
    }

    public LocalDate getBirthday() {
    	return birthday;
    }

    public void setBirthday(LocalDate birthday) {
    	this.birthday = birthday;
    }

    public String getJob() {
    	return job;
    }

    public void setJob(String job) {
    	this.job = job;
    }

    @Override
    public String toString() {
    	return "Person{" +
    		"id=" + id +
    		", name='" + name + "\'"
    		", age=" + age +
    		"}";
    }
}
```

### Getter와 Setter를 Lombok으로 변환

```java
@Entity
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @Getter
    @Setter
    private String name;

	@Getter
    @Setter
    private int age;

	@Getter
    @Setter
    private String hobby;

    @Getter
    @Setter
    private String bloodType;

    @Getter
    @Setter
    private String address;

    @Getter
    @Setter
    private LocalDate birthday;

	@Getter
    @Setter
    private String job;

    @Override
    public String toString() {
    	return "Person{" +
    		"id=" + id +
    		", name='" + name + "\'"
    		", age=" + age +
    		"}";
    }
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
    	person.setBloodType("A");

    	personRepository.save(person);

//    	System.out.println(personRepository.findAll());
    	List<Person> people = personRepository.findAll();

    	assertThat(people.size()).isEqualTo(1);
    	assertThat(people.get(0).getName()).isEqualTo("martin");
    	assertThat(people.get(0).getAge()).isEqualTo(10);
    	assertThat(people.get(0).getBloodType()).isEqualTo("A");
    }
}
```

#### @Getter와 @Setter를 Class Scope에서 사용하기

```java
@Entity
@Getter
@Setter
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @Override
    public String toString() {
    	return "Person{" +
    		"id=" + id +
    		", name='" + name + "\'"
    		", age=" + age +
    		"}";
    }
}
```

### @ToString 사용하기

```java
@Entity
@Getter
@Setter
@ToString
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    private String phoneNumber;
}
```

#### 특정 필드를 @ToString에서 제외하기

```java
@Entity
@Getter
@Setter
@ToString(exclude = "phoneNumber")
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    private String phoneNumber;
}
```

```java
@Entity
@Getter
@Setter
@ToString
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;
}
```

## Lombok (2)

### Lombok Constructor의 필요성 확인

```java
@Entity
@Getter
@Setter
@ToString
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;

    public Person(String name, int age) {
    	this.name = name;
    	this.age = age;
    }

    public Person() {

    }

    public Person(String name, int age, String hobby, String bloodType, String address, LocalDate birthday, String job, String phoneNumber) {
    	this.name = name;
    	this.age = age;
    	this.hobby = hobby;
    	this.bloodType = bloodType;
    	this.address = address;
    	this.birthday = birthday;
    	this.job = job;
    	this.phoneNumber = phoneNumber;
    }
}
```

### Lombok의 생성자

* @NoArgsConstructor
* @AllArgsConstructor
* @RequiredArgsConstructor : @NonNull

### Lombok에서 제공하는 Constructor 적용

```java
@Entity
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @NonNull
    private String name;

    @NonNull
    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;
}
```

```java
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void allArgsConstructor() {
    	Person person = new Person(1L, "martin", 10, "reading", "A", "분당", LocalDate.of(2019, 1, 1), "programmer", "010-1111-2222");
    }

    @Test
    void constructorTest() {
    	Person person = new Person("martin", 10);
    }
}
```

### HashCode And Equals

* HashCode : 같은 객체라는 것을 의미함
* Equals : 해당 객체가 같은 값을 가지고 있는 것을 의미함

```java
@Entity
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @NonNull
    private String name;

    @NonNull
    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;

    public boolean equals(Object object) {
    	if (object == null) {
    		return false;
    	}

    	Person person = (Person) object;

    	if (!person.getName().equals(this.getName())) {
    		return false;
    	}

    	if (person.getAge() != this.getAge()) {
    		return false;
    	}

    	return true;
    }

    public int hashCode() {
    	return (name + age).hashCode();
    }
}
```

```java
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void hashCodeAndEquals() {
    	Person person1 = new Person("martin", 10);
    	Person person2 = new Person("martin", 10);

    	System.out.println(person1.equals(person2));
    	System.out.println(person1.hashCode());
    	System.out.println(person2.hashCode());

    	Map<Person, Integer> map = new HashMap<>();
    	map.put(person1, person1.getAge());

    	System.out.println(map);
    	System.out.println(map.get(person2));
    }
}
```


### Lombok에서 제공하는 @EqualsAndHashCode 적용

```java
@Entity
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @NonNull
    private String name;

    @NonNull
    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;
}
```

### Lombok에서 제공하는 @Data은?

* @Getter
* @Setter
* @RequiredArgsConstructor
* @ToString
* @EqualsAndHashCode

### @Data의 적용

```java
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Person {
    @Id
    @GeneratedValue
    private Long id;

    @NonNull
    private String name;

    @NonNull
    private int age;

    private String hobby;

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;
}
```

