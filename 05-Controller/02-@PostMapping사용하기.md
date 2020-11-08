# @PostMapping 사용하기

## 이론

* @PostMapping
  * Post 메소드의 Http 요청을 받을 수 있는 메소드임을 명시하는 어노테이션
* @RequestBody
  * Request Body에 있는 데이터를 읽어서 파라미터로 매핑할 수 있도록 도와줌
* @ResponseStatus
  * http 응답에 대한 코드값을 지정한 값으로 변경할 수 있음

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
}
```
