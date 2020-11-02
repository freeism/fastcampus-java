# @PutMapping 사용하기

## 이론

### @PutMapping

* Put 메소드의 Http 요청을 받을 수 있는 메소드임을 명시하는 어노테이션

### @PatchMapping

* Patch 메소드의 Http 요청을 받을 수 있는 메소드임을 명시하는 어노테이션

## 실전코드

```java
@RequestMapping(value = "/api/person")
@RestController
@Slf4j
public class PersonController {
    @Autowired
    private PersonService personService;
    @Autowired
    private PersonRepository personRepository;

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        return personService.getPerson(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void postPerson(@RequestBody Person person) {
        personService.put(person);

        log.info("person -> {} ", personRepository.findAll());
    }

    @PutMapping("/{id}")
    public void modifyPerson(@PathVariable Long id, @RequestBody PersonDto personDto) {
        personService.modify(id, personDto);

        log.info("person -> {} ", personRepository.findAll());
    }

    @PatchMapping("/{id}")
    public void modifyPerson(@PathVariable Long id, String name) {
        personService.modify(id, name);

        log.info("person -> {} ", personRepository.findAll());
    }
}
```

```java
@Data
public class PersonDto {
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
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
@Data
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NonNull
    @NotEmpty
    @Column(nullable = false)
    private String name;

    @NonNull
    @Min(1)
    private int age;

    private String hobby;

    @NotEmpty
    @NonNull
    @Column(nullable = false)
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

    public void set(PersonDto personDto) {
        if (personDto.getAge() != 0) {
            this.setAge(personDto.getAge());
        }

        if (!StringUtils.isEmpty(personDto.getHobby())) {
            this.setHobby(personDto.getHobby());
        }

        if (!StringUtils.isEmpty(personDto.getBloodType())) {
            this.setBloodType(personDto.getBloodType());
        }

        if (!StringUtils.isEmpty(personDto.getAddress())) {
            this.setAddress(personDto.getAddress());
        }

        if (!StringUtils.isEmpty(personDto.getJob())) {
            this.setJob(personDto.getJob());
        }

        if (!StringUtils.isEmpty(personDto.getPhoneNumber())) {
            this.setPhoneNumber(personDto.getPhoneNumber());
        }
    }
}
```

```java
@Service
@Slf4j
public class PersonService {
    @Autowired
    private PersonRepository personRepository;

    public List<Person> getPeopleExcludeBlocks() {
        return personRepository.findByBlockIsNull();
    }

    public List<Person> getPeopleByName(String name) {
        return personRepository.findByName(name);
    }

    @Transactional(readOnly = true)
    public Person getPerson(Long id) {
        Person person = personRepository.findById(id).orElse(null);

        log.info("person : {}", person);

        return person;
    }

    @Transactional
    public void put(Person person) {
        personRepository.save(person);
    }

    @Transactional
    public void modify(Long id, PersonDto personDto) {
        Person person = personRepository.findById(id).orElseThrow(() -> new RuntimeException("아이디가 존재하지 않습니다."));

        if (!person.getName().equals(personDto.getName())) {
            throw new RuntimeException("이름이 다릅니다.");
        }

        person.set(personDto);

        personRepository.save(person);
    }

    @Transactional
    public void modify(Long id, String name) {
        Person person = personRepository.findById(id).orElseThrow(() -> new RuntimeException("아이디가 존재하지 않습니다."));

        person.setName(name);

        personRepository.save(person);
    }
}
```

```java
@SpringBootTest
class PersonControllerTest {
    @Autowired
    private PersonController personController;

    private MockMvc mockMvc;

    @Test
    void getPerson() throws Exception {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).build();

        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/person/1"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("martin"));
    }

    @Test
    void postPerson() throws Exception {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).build();

        mockMvc.perform(
            MockMvcRequestBuilders.post("/api/person")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content("{\n"
                    + "  \"name\": \"martin2\",\n"
                    + "  \"age\": 20,\n"
                    + "  \"bloodType\": \"A\"\n"
                    + "}"))
            .andDo(print())
            .andExpect(status().isCreated());
    }

    @Test
    void modifyPerson() throws Exception {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).build();

        mockMvc.perform(
            MockMvcRequestBuilders.put("/api/person/1")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content("{\n"
                    + "  \"name\": \"martin\",\n"
                    + "  \"age\": 20,\n"
                    + "  \"bloodType\": \"A\"\n"
                    + "}"))
            .andDo(print())
            .andExpect(status().isOk());
    }

    @Test
    void modifyName() throws Exception {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).build();

        mockMvc.perform(
            MockMvcRequestBuilders.patch("/api/person/1")
                .param("name", "martin22"))
            .andDo(print())
            .andExpect(status().isOk());
    }
}
```

```sql
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`, `job`) values (1, 'martin', 10, 'A', 1991, 8, 15, 'programmer');
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (2, 'david', 9, 'B', 1992, 7, 21);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (3, 'dennis', 8, 'O', 1993, 10, 15);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (4, 'sophia', 7, 'AB', 1994, 8, 31);
insert into person(`id`, `name`, `age`, `blood_type`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (5, 'benny', 6, 'A', 1995, 12, 23);

insert into block(`id`, `name`) values (1, 'dennis');
insert into block(`id`, `name`) values (2, 'sophia');

update person set block_id = 1 where id = 3;
update person set block_id = 2 where id = 4;
```
