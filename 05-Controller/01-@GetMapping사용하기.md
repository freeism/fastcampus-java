# @GetMapping 사용하기

## 이론

* @GetMapping
  * Get 메소드의 Http 요청을 받을 수 있는 메소드임을 명시하는 어노테이션
* @PathVariable
  * Rest의 Url의 값을 읽어서 메소드의 파라미터로 매핑시킬 수 있도록 도와줌
  * {id}로 표기하면, 해당 위치에 들어오는 문자열을 id 파라미터에 할당해줌

## 실전코드

```java
@RequestMapping(value = "/api/person")
@RestController
public class PersonController {
    @Autowired
    private PersonService personService;

    @GetMapping
    @RequestMapping(value = "/{id}")
    public Person getPerson(@PathVariable Long id) {
        return personService.getPerson(id);
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
}
```

```http request
GET http://localhost:8080/api/person/1
```
