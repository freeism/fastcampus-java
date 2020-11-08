# JPA @Query

## 이론

* @Embedded
  * 다른 객체를 Entity의 속성으로 가져옴
* @Embeddable
  * 자기 객체를 다른 Entity의 속성으로 사용할 수 있음
* @Query
  * Naming 컨벤션에 따라 생성되는 쿼리가 아니라, 커스터마이징된 쿼리를 직접 지정하여 생성함
  * ?1 (숫자)를 통해서 parameter를 순서대로 주입하여 사용할 수 있음
  * @Param("A")으로 parameter를 받게 되면, :A로 파라미터를 주입하여 사용할 수 있음
  * `nativeQuery=true`를 사용하게 되면, JPQL이 아니라 써준 그대로 네이티브 쿼리를 생성해줌
* @Valid
  * 해당 객체의 유효성을 검토하겠다는 것을 의미함
* @Min
  * 숫자관련 필드에서 최소값은 얼마인지 정의함
* @Max
  * 숫자관련 필드에서 최대값은 얼마인지 정의함

## 실전코드

```java
@Embeddable
@Data
@NoArgsConstructor
public class Birthday {
    private int yearOfBirthday;

    @Min(1)
    @Max(12)
    private int monthOfBirthday;

    @Min(1)
    @Max(31)
    private int dayOfBirthday;

    public Birthday(LocalDate birthday) {
        this.yearOfBirthday = birthday.getYear();
        this.monthOfBirthday = birthday.getMonthValue();
        this.dayOfBirthday = birthday.getDayOfMonth();
    }
}
```

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

    @Valid
    @Embedded
    private Birthday birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;

    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
    @ToString.Exclude
    private Block block;
}
```

```java
public interface PersonRepository extends JpaRepository<Person, Long> {
    List<Person> findByName(String name);

    List<Person> findByBlockIsNull();

    List<Person> findByBloodType(String bloodType);

    @Query(value = "select person from Person person where person.birthday.monthOfBirthday = :monthOfBirthday")
    List<Person> findByMonthOfBirthday(@Param("monthOfBirthday") int monthOfBirthday);
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

    @Test
    void findByBloodType() {
        givenPerson("martin", 10, "A");
        givenPerson("david", 9, "B");
        givenPerson("dennis", 8, "O");
        givenPerson("sophia", 7, "AB");
        givenPerson("benny", 6, "A");
        givenPerson("john", 5, "A");

        List<Person> result = personRepository.findByBloodType("A");

        result.forEach(System.out::println);
    }

    @Test
    void findByBirthdayBetween() {
        givenPerson("martin", 10, "A", LocalDate.of(1991, 8, 15));
        givenPerson("david", 9, "B", LocalDate.of(1992, 7, 10));
        givenPerson("dennis", 8, "O", LocalDate.of(1993, 1, 5));
        givenPerson("sophia", 7, "AB", LocalDate.of(1994, 6, 30));
        givenPerson("benny", 6, "A", LocalDate.of(1995, 8, 30));

        List<Person> result = personRepository.findByMonthOfBirthday(8);

        result.forEach(System.out::println);
    }

    private void givenPerson(String name, int age, String bloodType) {
        givenPerson(name, age, bloodType, null);
    }

    private void givenPerson(String name, int age, String bloodType, LocalDate birthday) {
        Person person = new Person(name, age, bloodType);
        person.setBirthday(new Birthday(birthday));

        personRepository.save(person);
    }
}
```
