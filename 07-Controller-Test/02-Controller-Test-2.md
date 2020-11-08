# Controller Test #2

## 이론

* Controller Test에서 사용하는 메소드
    * jsonPath
      * $ : 객체를 의미함
      * .name : 객체의 name attribute를 가져옴, getName()을 사용한다고 생각하면 됨
      * . 체이닝을 통해서 recursive하게 데이터 추출이 가능
    * value(A)
      * 값이 A와 동일한지 검증함
    * imEmpty()
      * 값이 빈 값인지 검증함
    * isNumber()
      * 값이 숫자값인지 검증함
    * isBoolean()
      * true/false 값인지 검증함
* JsonSerializer<T>
  * serialize()
* @Configuration
  * Spring의 Configuration Bean임을 표시하는 어노테이션
* MappingJackson2HttpMessageConverter
  * Spring의 Controller에서 반환하는 결과를 Json으로 만드는데, 해당 포맷을 커스터마이징할 수 있게 해줌
* SimpleModule
  * 커스텀으로 만든 Serializer를 Converter에 등록하기 위해서 모듈화해줌

## 실전코드

```java
@Configuration
public class JsonConfig {
    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter(ObjectMapper objectMapper) {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        converter.setObjectMapper(objectMapper);

        return converter;
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new BirthdayModule());
        objectMapper.registerModule(new JavaTimeModule());

        objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);

        return objectMapper;
    }

    static class BirthdayModule extends SimpleModule {
        BirthdayModule() {
            super();
            addSerializer(Birthday.class, new BirthdaySerializer());
        }
    }
}
```

```java
public class BirthdaySerializer extends JsonSerializer<Birthday> {
    @Override
    public void serialize(Birthday value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        if (value != null) {
            gen.writeObject(LocalDate.of(value.getYearOfBirthday(), value.getMonthOfBirthday(), value.getDayOfBirthday()));
        }
    }
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
    @Autowired
    private MappingJackson2HttpMessageConverter messageConverter;

    private MockMvc mockMvc;

    @BeforeEach
    void beforeEach() {
        mockMvc = MockMvcBuilders.standaloneSetup(personController).setMessageConverters(messageConverter).build();
    }

    @Test
    void getPerson() throws Exception {
        mockMvc.perform(
            MockMvcRequestBuilders.get("/api/person/1"))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("martin"))
            .andExpect(jsonPath("hobby").isEmpty())
            .andExpect(jsonPath("address").isEmpty())
            .andExpect(jsonPath("$.birthday").value("1991-08-15"))
            .andExpect(jsonPath("$.job").isEmpty())
            .andExpect(jsonPath("$.phoneNumber").isEmpty())
            .andExpect(jsonPath("$.deleted").value(false))
            .andExpect(jsonPath("$.age").isNumber())
            .andExpect(jsonPath("$.birthdayToday").isBoolean());
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
