# JPA Relation #3

## 이론

* cascade
  * 관련된 entity의 영속성을 함께 관리할 수 있도록 해줌
  * CascadeType.PERSIST
    * insert할 경우 관련 entity도 함께 insert함
  * CascadeType.MERGE 
    * update할 경우 관련 entity도 함께 update함
  * CascadeType.REMOVE 
    * delete할 경우 관련 entity도 함께 delete함
  * CascadeType.ALL
    * 모든 케이스에 대해 영속성을 함께 관리함
* orphanRemoval
  * 관련 entity의 relation이 사라질 때, entity를 함께 삭제해줌
* fetch
  * FetchType.EAGER 
    * 항상 relation이 있는 entity를 join하여 값을 함께 가져옴
  * FetchType.LAZY 
    * 해당 객체가 필요한 시점에 id를 통해 새로 select해서 값을 가져옴

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

    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
    @ToString.Exclude
    private Block block;
}
```

```java
@Service
@Slf4j
public class PersonService {
    @Autowired
    private PersonRepository personRepository;

    public List<Person> getPeopleExcludeBlocks() {
        List<Person> people = personRepository.findAll();

        return people.stream().filter(person -> person.getBlock() == null).collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public Person getPerson(Long id) {
        Person person = personRepository.findById(id).get();

        log.info("person : {}", person);

        return person;
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
    @Autowired
    private BlockRepository blockRepository;

    @Test
    void getPeopleExcludeBlocks() {
        givenPeople();

        List<Person> result = personService.getPeopleExcludeBlocks();

        result.forEach(System.out::println);
    }

    @Test
    void cascadeTest() {
        givenPeople();

        List<Person> result = personRepository.findAll();
        result.forEach(System.out::println);

        Person person = result.get(3);
        person.getBlock().setStartDate(LocalDate.now());
        person.getBlock().setEndDate(LocalDate.now());

        personRepository.save(person);
        personRepository.findAll().forEach(System.out::println);

//        personRepository.delete(person);
//        personRepository.findAll().forEach(System.out::println);
//        blockRepository.findAll().forEach(System.out::println);

        person.setBlock(null);
        personRepository.save(person);
        personRepository.findAll().forEach(System.out::println);
        blockRepository.findAll().forEach(System.out::println);
    }

    @Test
    void getPerson() {
        givenPeople();

        Person person = personService.getPerson(3L);

        System.out.println(person);
    }

    private void givenPeople() {
        givenPerson("martin", 10, "A");
        givenPerson("david", 9, "B");
        givenBlockPerson("dennis",7,"O");
        givenBlockPerson("martin", 11, "AB");
    }

    private void givenPerson(String name, int age, String bloodType) {
        personRepository.save(new Person(name, age, bloodType));
    }

    private void givenBlockPerson(String name, int age, String bloodType) {
        Person blockPerson = new Person(name, age, bloodType);
        blockPerson.setBlock(new Block(name));

        personRepository.save(blockPerson);
    }
}
```
