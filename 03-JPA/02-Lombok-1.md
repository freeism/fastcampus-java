# Lombok #1

## 이론

* Lombok의 필요성
  * 자바 코드에서 필요한 기본 메소드들을 자동으로 생성해줌
  * 특히, entity는 반복적인 생성자, Getter, Setter들을 많이 생성하게 되어 Lombok이 유용함
* @Getter
  * getter는 class의 field variable의 값에 접근할 수 있는 메소드임
  * @Getter를 선언하면 기본 getter를 생성해줌
  * @Getter는 각 field variable에 선언할 수도 있고, class 상단에 선언할 수도 있음
* @Setter
  * setter는 class의 field variable의 값을 저장할 수 있는 메소드임
  * @Setter를 선언하면 기본 setter을 생성ㅇ해줌
  * @Setter는 각 field variable에 선언할 수도 있고, class 상단에 선언할 수도 있음
* @ToString
  * toString()은 기본적으로 해당 객체를 출력할 수 있도록 최상위 Object 객체에 선언되어 있음
  * toString()을 override하여 자신이 원하는 값을 출력할 수 있도록 커스터마이징 할 수 있음
  * Object 객체의 toString()은 해당 객체의 해쉬값을 출력하기 때문에, 일반적으로 toString()은 override 해야만 함
  * @ToString을 사용하면, 일반적으로 사용하는 방식의 toString()을 자동으로 생성해줌

## 실전코드

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

        System.out.println(personRepository.findAll());

        List<Person> people = personRepository.findAll();

        assertThat(people.size()).isEqualTo(1);
        assertThat(people.get(0).getName()).isEqualTo("martin");
        assertThat(people.get(0).getAge()).isEqualTo(10);
        assertThat(people.get(0).getBloodType()).isEqualTo("A");
    }
}
```
