# JPA data.sql 사용하기

## 이론

* data.sql
  * spring context를 실행시킬 때, data.sql에 적혀있는 쿼리를 실행시킴
  * main/resources에 선언하게 되면, 서버를 시작할 때 쿼리가 실행됨
  * test/resources에 선언하게 되면, 테스트를 실행할 때 쿼리가 실행됨

## 실전코드

```java
@Embeddable
@Data
@NoArgsConstructor
public class Birthday {
    private Integer yearOfBirthday;

    @Min(1)
    @Max(12)
    private Integer monthOfBirthday;

    @Min(1)
    @Max(31)
    private Integer dayOfBirthday;

    public Birthday(LocalDate birthday) {
        this.yearOfBirthday = birthday.getYear();
        this.monthOfBirthday = birthday.getMonthValue();
        this.dayOfBirthday = birthday.getDayOfMonth();
    }
}
```

```java
@Entity
@Data
@NoArgsConstructor
@RequiredArgsConstructor
public class Block {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NonNull
    private String name;

    private String reason;

    private LocalDate startDate;

    private LocalDate endDate;
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
    @GeneratedValue(strategy = GenerationType.IDENTITY)
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
@SpringBootTest
class BlockRepositoryTest {
    @Autowired
    private BlockRepository blockRepository;

    @Test
    void crud() {
        Block block = new Block();
        block.setName("martin");
        block.setReason("친하지않아서");
        block.setStartDate(LocalDate.now());
        block.setEndDate(LocalDate.now());

        blockRepository.save(block);

        List<Block> blocks = blockRepository.findAll();

        assertThat(blocks.size()).isEqualTo(3);
        assertThat(blocks.get(0).getName()).isEqualTo("dennis");
        assertThat(blocks.get(1).getName()).isEqualTo("sophia");
        assertThat(blocks.get(2).getName()).isEqualTo("martin");
    }
}
```

```java
@Transactional
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void crud() {
        Person person = new Person();
        person.setName("john");
        person.setAge(10);
        person.setBloodType("A");

        personRepository.save(person);

        List<Person> result = personRepository.findByName("john");

        assertThat(result.size()).isEqualTo(1);
        assertThat(result.get(0).getName()).isEqualTo("john");
        assertThat(result.get(0).getAge()).isEqualTo(10);
        assertThat(result.get(0).getBloodType()).isEqualTo("A");
    }

    @Test
    void findByBloodType() {
        List<Person> result = personRepository.findByBloodType("A");

        assertThat(result.size()).isEqualTo(2);
        assertThat(result.get(0).getName()).isEqualTo("martin");
        assertThat(result.get(1).getName()).isEqualTo("benny");
    }

    @Test
    void findByBirthdayBetween() {
        List<Person> result = personRepository.findByMonthOfBirthday(8);

        assertThat(result.size()).isEqualTo(2);
        assertThat(result.get(0).getName()).isEqualTo("martin");
        assertThat(result.get(1).getName()).isEqualTo("sophia");
    }
}
```

```java
@SpringBootTest
class PersonServiceTest {
    @Autowired
    private PersonService personService;
    @Autowired
    private PersonRepository personRepository;

    @Test
    void getPeopleExcludeBlocks() {
        List<Person> result = personService.getPeopleExcludeBlocks();

        personRepository.findAll().forEach(System.out::println);

        assertThat(result.size()).isEqualTo(3);
        assertThat(result.get(0).getName()).isEqualTo("martin");
        assertThat(result.get(1).getName()).isEqualTo("david");
        assertThat(result.get(2).getName()).isEqualTo("benny");
    }

    @Test
    void getPeopleByName() {
        List<Person> result = personService.getPeopleByName("martin");

        assertThat(result.size()).isEqualTo(1);
        assertThat(result.get(0).getName()).isEqualTo("martin");
    }

    @Test
    void getPerson() {
        Person person = personService.getPerson(3L);

        assertThat(person.getName()).isEqualTo("dennis");
    }
}
```

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class MycontactApplicationTests {

  @Test
  public void contextLoads() {
  }

}
```

```sql
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (1, 'martin', 10, 'A', 1991, 8, 15);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (2, 'david', 9, 'B', 1992, 7, 21);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (3, 'dennis', 8, 'O', 1993, 10, 15);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (4, 'sophia', 7, 'AB', 1994, 8, 31);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (5, 'benny', 6, 'A', 1995, 12, 23);

insert into block(`id`, `name`) values (1, 'dennis');
insert into block(`id`, `name`) values (2, 'sophia');

update person set block_id = 1 where id = 3;
update person set block_id = 2 where id = 4;
```
