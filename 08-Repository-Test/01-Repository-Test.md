# Repository Test

## 실전코드

```java
@Transactional
@SpringBootTest
class PersonRepositoryTest {
    @Autowired
    private PersonRepository personRepository;

    @Test
    void findByName() {
        List<Person> people = personRepository.findByName("tony");
        assertThat(people.size()).isEqualTo(1);

        Person person = people.get(0);
        assertAll(
            () -> assertThat(person.getName()).isEqualTo("tony"),
            () -> assertThat(person.getHobby()).isEqualTo("reading"),
            () -> assertThat(person.getAddress()).isEqualTo("서울"),
            () -> assertThat(person.getBirthday()).isEqualTo(Birthday.of(LocalDate.of(1991, 7, 10))),
            () -> assertThat(person.getJob()).isEqualTo("officer"),
            () -> assertThat(person.getPhoneNumber()).isEqualTo("010-2222-5555"),
            () -> assertThat(person.isDeleted()).isEqualTo(false)
        );
    }

    @Test
    void findByNameIfDeleted() {
        List<Person> people = personRepository.findByName("andrew");

        assertThat(people.size()).isEqualTo(0);
    }

    @Test
    void findByMonthOfBirthday() {
        List<Person> people = personRepository.findByMonthOfBirthday(7);

        assertThat(people.size()).isEqualTo(2);
        assertAll(
            () -> assertThat(people.get(0).getName()).isEqualTo("david"),
            () -> assertThat(people.get(1).getName()).isEqualTo("tony")
        );
    }

    @Test
    void findPeopleDeleted() {
        List<Person> people = personRepository.findPeopleDeleted();

        assertThat(people.size()).isEqualTo(1);
        assertThat(people.get(0).getName()).isEqualTo("andrew");
    }
}
```

```sql
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (1, 'martin', 1991, 8, 15);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (2, 'david', 1992, 7, 21);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (3, 'dennis', 1993, 10, 15);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (4, 'sophia', 1994, 8, 31);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (5, 'benny', 1995, 12, 23);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`, `job`, `hobby`, `phone_number`, `address`)
    values (6, 'tony', 1991, 7, 10, 'officer', 'reading', '010-2222-5555', '서울');
insert into person(`id`, `name`, `deleted`) values (7, 'andrew', true);
```
