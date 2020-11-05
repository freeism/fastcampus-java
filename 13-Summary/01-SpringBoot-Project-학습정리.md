# Summary

## SpringBoot Project 학습정리

### SpringBoot 특성

* 스프링의 생산성
* Coding By Convention 활용 (CoC : Convention over Configuration)

### 학습했던 내용 정리

* 스프링부트 프로젝트 생성
* Gradle을 이용한 의존성 관리
* Iteration(반복주기) 개발로 2-cycle 개발 진행

#### 1-cycle 내용정리

* JPA
  * Entity 생성
  * @OneToOne Relation
    * CascadeType
    * FetchType
    * Optional, orphanRemoval
  * QueryMethod
  * @Embedded
  * @Valid
  * @Query
  * @Where (for soft-delete)
  * Data.sql

* SpringMvc
  * @GetMapping
  * @PostMapping
  * @PutMapping
  * @PatchMapping
  * @DeleteMapping
  * @PathVariable
  * @RequestBody

* Lombok
  * @Getter
  * @Setter
  * @ToString
  * @Constructor
  * @EqualsAndHashCode
  * @Data

* SpringTest

* Java8
  * Stream
  * Fileter
  * Map

#### 2-cycle 내용정리

* SpringMvc
  * CustomJsonSerializer

* SpringTest
  * MockMvc Test
  * Matcher
  * Junit5

* MockTest
  * Mockito
  * CustomArgumentMatcher

* Exception Handling
  * CustomException
  * ExceptionalHandler
  * GlobalExceptionHandler

* Parameter Validator
  * @NotEmpty
  * @NotBlank
  * @Valid

* Paging
  * Pageable
  * Page<T>

### 추가로 학습할 것

* FrontEnd 개발
  * Web
    * VueJs
    * ReactJs
  * App
    * Android App
    * IOS App

* DB 연동
  * MySQL
  * MongDB

* Spring(Boot)의 중급활용
  * 추가적인 설정
  * Customizing 설정

* JPA 중급활용
  * 다양한 Relation
  * QueryDSL
  * Jooq

* 로직의 확장
  * 추가적인 스펙

```java
@Entity
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
@Data
public class Group {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	private String description;

	@OneToMany
	private List<Person> personList;
}
```

```java
@RequestMapping(value = "/api/group")
@RestController
public class GroupController {
	@GetMapping
	public List<Group getAll() {
		return null;	// Group 전체 정보 가져오기
	}

	@GetMapping("/{id}")
	public Group getGroup(@PathVariable Long id) {
		return null;	// 특정 Group 정보 가져오기
	}

	@PostMapping
	@ResponseStatus(HttpStatus.CREATED)
	public void postGroup(@RequestBody Object groupDto) {	// DTO 구현필요
		// Group 생성하기
	}

	@PatchMapping("/{id}")
	public void modifyGroup(@PathVariable Long id, String description) {
		// Group description 수정하기
	}

	@GetMapping("/{id}/people")
	public List<Person getPeopleInGroup(@PathVariable Long id) {
		return null;	// 특정 그룹의 Person 리스트 가져오기
	}

	@PutMapping("/{id}/person/{personId}")
	public void putPersonInGroup(@PathVariable Long id, @PathVariable Long personId) {
		// Person 정보를 Group 정보에 매핑하기
	}
}
```
