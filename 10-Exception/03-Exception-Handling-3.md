# Exception Handling #3

## 이론

### 

*  

## 실전코드

```java
@RestController
public class HelloWorldController {
    @GetMapping(value = "/api/helloWorld")
    public String helloWorld() {
        return "HelloWorld";
    }

    @GetMapping(value = "/api/helloException")
    public String helloException() {
        throw new RuntimeException("Hello RuntimeException");
    }
}
```

```java
@RequestMapping(value = "/api/person")
@RestController
@Slf4j
public class PersonController {
    @Autowired
    private PersonService personService;

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        return personService.getPerson(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void postPerson(@RequestBody PersonDto personDto) {
        personService.put(personDto);
    }

    @PutMapping("/{id}")
    public void modifyPerson(@PathVariable Long id, @RequestBody PersonDto personDto) {
        personService.modify(id, personDto);
    }

    @PatchMapping("/{id}")
    public void modifyPerson(@PathVariable Long id, String name) {
        personService.modify(id, name);
    }

    @DeleteMapping("/{id}")
    public void deletePerson(@PathVariable Long id) {
        personService.delete(id);
    }
}
```

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(RenameIsNotPermittedException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleRenameNoPermittedException(RenameIsNotPermittedException ex) {
        return ErrorResponse.of(HttpStatus.BAD_REQUEST, ex.getMessage());
    }

    @ExceptionHandler(PersonNotFoundException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handlePersonNotFoundException(PersonNotFoundException ex) {
        return ErrorResponse.of(HttpStatus.BAD_REQUEST, ex.getMessage());
    }

    @ExceptionHandler(RuntimeException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleRuntimeException(RuntimeException ex) {
        log.error("서버오류 : {}", ex.getMessage(), ex);
        return ErrorResponse.of(HttpStatus.INTERNAL_SERVER_ERROR, "알 수 없는 서버 오류가 발생하였습니다");
    }
}
```

```java
@SpringBootTest
class HelloWorldControllerTest {
    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void beforeEach() {
        mockMvc = MockMvcBuilders
            .webAppContextSetup(wac)
            .alwaysDo(print())
            .build();
    }

    @Test
    void helloWorld() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/helloWorld"))
            .andExpect(status().isOk())
            .andExpect(MockMvcResultMatchers.content().string("HelloWorld"));
    }

    @Test
    void helloException() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/helloException"))
            .andExpect(status().isInternalServerError())
            .andExpect(jsonPath("$.code").value(500))
            .andExpect(jsonPath("$.message").value("알 수 없는 서버 오류가 발생하였습니다"));
    }
}
```

```java
@Slf4j
@SpringBootTest
@Transactional
class PersonControllerTest {
    @Autowired
    private PersonRepository personRepository;
    @Autowired
    private ObjectMapper objectMapper;
    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void beforeEach() {
        mockMvc = MockMvcBuilders
            .webAppContextSetup(wac)
            .alwaysDo(print())
            .build();
    }

    @Test
    void getPerson() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/person/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("martin"))
            .andExpect(jsonPath("$.hobby").isEmpty())
            .andExpect(jsonPath("$.address").isEmpty())
            .andExpect(jsonPath("$.birthday").value("1991-08-15"))
            .andExpect(jsonPath("$.job").isEmpty())
            .andExpect(jsonPath("$.phoneNumber").isEmpty())
            .andExpect(jsonPath("$.deleted").value(false))
            .andExpect(jsonPath("$.age").isNumber())
            .andExpect(jsonPath("$.birthdayToday").isBoolean());
    }

    @Test
    void postPerson() throws Exception {
        PersonDto dto = PersonDto.of("martin", "programming", "판교", LocalDate.now(), "programmer", "010-1111-2222");

        mockMvc.perform(
            MockMvcRequestBuilders.post("/api/person")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andExpect(status().isCreated());

        Person result = personRepository.findAll(Sort.by(Direction.DESC, "id")).get(0);

        assertAll(
            () -> assertThat(result.getName()).isEqualTo("martin"),
            () -> assertThat(result.getHobby()).isEqualTo("programming"),
            () -> assertThat(result.getAddress()).isEqualTo("판교"),
            () -> assertThat(result.getBirthday()).isEqualTo(Birthday.of(LocalDate.now())),
            () -> assertThat(result.getJob()).isEqualTo("programmer"),
            () -> assertThat(result.getPhoneNumber()).isEqualTo("010-1111-2222")
        );
    }

    @Test
    void postPersonIfNameIsNull() throws Exception {
        PersonDto dto = new PersonDto();

        mockMvc.perform(
            MockMvcRequestBuilders.post("/api/person")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andExpect(jsonPath("$.code").value(500))
            .andExpect(jsonPath("$.message").value("알 수 없는 서버 오류가 발생하였습니다"));
    }

    @Test
    void modifyPerson() throws Exception {
        PersonDto dto = PersonDto.of("martin", "programming", "판교", LocalDate.now(), "programmer", "010-1111-2222");

        mockMvc.perform(
            MockMvcRequestBuilders.put("/api/person/1")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andExpect(status().isOk());

        Person result = personRepository.findById(1L).get();

        assertAll(
            () -> assertThat(result.getName()).isEqualTo("martin"),
            () -> assertThat(result.getHobby()).isEqualTo("programming"),
            () -> assertThat(result.getAddress()).isEqualTo("판교"),
            () -> assertThat(result.getBirthday()).isEqualTo(Birthday.of(LocalDate.now())),
            () -> assertThat(result.getJob()).isEqualTo("programmer"),
            () ->assertThat(result.getPhoneNumber()).isEqualTo("010-1111-2222")
        );
    }

    @Test
    void modifyPersonIfNameIsDifferent() throws Exception {
        PersonDto dto = PersonDto.of("james", "programming", "판교", LocalDate.now(), "programmer", "010-1111-2222");

        mockMvc.perform(
            MockMvcRequestBuilders.put("/api/person/1")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value(400))
            .andExpect(jsonPath("$.message").value("이름 변경이 허용되지 않습니다"));
    }

    @Test
    void modifyPersonIfPersonNotFound() throws Exception {
        PersonDto dto = PersonDto.of("martin", "programming", "판교", LocalDate.now(), "programmer", "010-1111-2222");

        mockMvc.perform(
            MockMvcRequestBuilders.put("/api/person/10")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value(400))
            .andExpect(jsonPath("$.message").value("Person Entity가 존재하지 않습니다"));
    }

    @Test
    void modifyName() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.patch("/api/person/1")
                .param("name", "martinModified"))
            .andExpect(status().isOk());

        assertThat(personRepository.findById(1L).get().getName()).isEqualTo("martinModified");
    }

    @Test
    void deletePerson() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.delete("/api/person/1"))
            .andExpect(status().isOk());

        assertTrue(personRepository.findPeopleDeleted().stream().anyMatch(person -> person.getId().equals(1L)));
    }

    private String toJsonString(PersonDto personDto) throws JsonProcessingException {
        return objectMapper.writeValueAsString(personDto);
    }
}
```
