# Controller Test #1

## 이론

### 

*  

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
@Data
@NoArgsConstructor
@AllArgsConstructor(staticName = "of")
public class PersonDto {
    private String name;
    private String hobby;
    private String address;
    private LocalDate birthday;
    private String job;
    private String phoneNumber;
}
```

```java
@Slf4j
@SpringBootTest
@Transactional
class PersonControllerTest {
    @Autowired
    private PersonController personController;
    @Autowired
    private PersonRepository personRepository;
    @Autowired
    private ObjectMapper objectMapper;

    private MockMvc mockMvc;

    @BeforeEach
    void beforeEach() {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).build();
    }

    @Test
    void getPerson() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/person/1"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("martin"));
    }

    @Test
    void postPerson() throws Exception {
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
        PersonDto dto = PersonDto.of("martin", "programming", "판교", LocalDate.now(), "programmer", "010-1111-2222");

        mockMvc.perform(
            MockMvcRequestBuilders.put("/api/person/1")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(toJsonString(dto)))
            .andDo(print())
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

        assertThrows(NestedServletException.class, () ->
            mockMvc.perform(
                MockMvcRequestBuilders.put("/api/person/1")
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content(toJsonString(dto)))
                .andDo(print())
                .andExpect(status().isOk()));
    }

    @Test
    void modifyName() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.patch("/api/person/1")
                .param("name", "martinModified"))
            .andDo(print())
            .andExpect(status().isOk());

        assertThat(personRepository.findById(1L).get().getName()).isEqualTo("martinModified");
    }

    @Test
    void deletePerson() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.delete("/api/person/1"))
            .andDo(print())
            .andExpect(status().isOk());

        assertTrue(personRepository.findPeopleDeleted().stream().anyMatch(person -> person.getId().equals(1L)));
    }

    private String toJsonString(PersonDto personDto) throws JsonProcessingException {
        return objectMapper.writeValueAsString(personDto);
    }
}
```

```sql
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (1, 'martin', 1991, 8, 15);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (2, 'david', 1992, 7, 21);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (3, 'dennis', 1993, 10, 15);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (4, 'sophia', 1994, 8, 31);
insert into person(`id`, `name`, `year_of_birthday`, `month_of_birthday`, `day_of_birthday`) values (5, 'benny', 1995, 12, 23);
```
