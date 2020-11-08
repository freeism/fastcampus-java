# Lombok #2

## 이론

* @NoArgsConstructor
  * 아무런 파라미터를 가지지 않은 생성자를 생성해줌
* @AllArgsConstructor
  * 전체 field variable을 파라미터로 가지는 생성자를 생성해줌
* @RequiredArgsConstructor
  * 필요한 field variable을 @NonNull로 선언하고, 해당 파라미터를 가지는 생성자를 생성해줌
* @EqualsAndHashCode
  * HashCode : 해시코드가 동일하면, 같은 객체라는 것을 의미함
  * Equal : 해당 객체가 같은 값을 가지고 있다는 것을 의미함
* @Data
  * @Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode를 한꺼번에 선언해줌

## 실전코드

```java
@Entity
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
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

    @NonNull
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

    @Test
    void hashCodeAndEquals() {
        Person person1 = new Person("martin", 10, "A");
        Person person2 = new Person("martin", 10, "A");

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
