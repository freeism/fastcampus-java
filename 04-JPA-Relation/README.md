# JPA-Relation

## JPA-Relation-1

### 차단기능을 추가해보기

```java
@Entity
@NoArgsConstructor
@AllArgsConstructor
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

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;

    private boolean block;

    private String blockReason;

    private LocalDate blockStartDate;

    private LocalDate blockEndDate;
}
```

### 신규 Entity를 생성해보기

```java
@Entity
@Data
public class Block {
	@Id
	@GeneratedValue
	private Long id;

	private String name;

	private String reason;

	private LocalDate startDate;

    private LocalDate endDate;
}
```

### 신규 Repository를 생성해보기

```java

public interface BlockRepository extends JpaRepository<Block, Long> {

}
```

### 생성된 Entity, Repository를 테스트해보기

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

		assertThat(blocks.size()).isEqualTo(1);
		assertThat(blocks.get(0).getName()).isEqualTo("martin");
	}
}
```

### Service를 생성해보기

```java
@Service
public class PersonService {
	@Autowired
	private PersonRepository PersonRepository;
	@Autowired
	private BlockRepository blockRepository;

	public List<Person> getPeopleExcludeBlocks() {
		List<Person> people = PersonRepository.findAll();
		List<Block> blocks = blockRepository.findAll();
		List<String> blockNames = blocks.stream().map(Block::getName).collect(Collectors.toList());

		return people.stream().filter(person -> !blockNames.contains(person.getName())).collect(Collectors.toList());
	}
}
```

### Service를 테스트해보기

```java
@SpringBootTest
class PersonServiceTest {
	@Autowired
	private PersonService PersonService;
	@Autowired
	private PersonRepository PersonRepository;
	@Autowired
	private BlockRepository blockRepository;

	@Test
	void getPeopleExcludeBlocks() {
		givenPeople();
		givenBlocks();

		List<Person> result = PersonService.getPeopleExcludeBlocks();

		// System.out.println(result);
		result.forEach(System.out::println);
	}

	private void givenPeople() {
		givenPerson("martin", 10, "A");
		givenPerson("david", 9, "B");
		givenPerson("dennis", 7, "O");
		givenPerson("martin", 11, "AB");
	}

	private void givenBlocks() {
		givenBlock("martin");
	}

	private void givenPerson(Sring name, int age, String bloodType) {
		PersonRepository.save(new Person(name, age, bloodType));
	}

	private void givenBlock(String name) {
		blockRepository.save(new Block(name));
	}
}
```

### Domain의 오류수정

```java
@Entity
@Data
@NoArgsConstructor
@RequiedArgsConstructor
public class Block {
	@Id
	@GeneratedValue
	private Long id;

	@NonNull
	private String name;

	private String reason;

	private LocalDate startDate;

    private LocalDate endDate;
}
```

### Person Domain에 Relation 추가하기

```java
@Entity
@NoArgsConstructor
@AllArgsConstructor
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

    private String bloodType;

    private String address;

    private LocalDate birthday;

    private String job;

    @ToString.Exclude
    private String phoneNumber;

    @OneToOne
    private Block block;
}
```

### Relation 추가하여 테스트 수정하기

```java
@SpringBootTest
class PersonServiceTest {
	@Autowired
	private PersonService PersonService;
	@Autowired
	private PersonRepository PersonRepository;
	@Autowired
	private BlockRepository blockRepository;

	@Test
	void getPeopleExcludeBlocks() {
		givenPeople();
		givenBlocks();

		List<Person> result = PersonService.getPeopleExcludeBlocks();

		// System.out.println(result);
		result.forEach(System.out::println);
	}

	private void givenPeople() {
		givenPerson("martin", 10, "A");
		givenPerson("david", 9, "B");
		givenPerson("dennis", 7, "O");
		givenBlockPerson("martin", 11, "AB");
	}

	private void givenBlocks() {
		givenBlock("martin");
	}

	private void givenPerson(Sring name, int age, String bloodType) {
		PersonRepository.save(new Person(name, age, bloodType));
	}

	private void givenBlockPerson(String name, int age, String bloodType) {
		Person blockPerson = new Person(name, age, bloodType);
		blockPerson.setBlock(givenBlock(name));

		PersonRepository.save(blockPerson);
	}

	private Block givenBlock(String name) {
		return blockRepository.save(new Block(name));
	}
}
```

### Service의 오류수정

```java
@Service
public class PersonService {
	@Autowired
	private PersonRepository PersonRepository;
	@Autowired
	private BlockRepository blockRepository;

	public List<Person> getPeopleExcludeBlocks() {
		List<Person> people = PersonRepository.findAll();
		// List<Block> blocks = blockRepository.findAll();
		// List<String> blockNames = blocks.stream().map(Block::getName).collect(Collectors.toList());

		return people.stream().filter(person -> person.getBlock() == null).collect(Collectors.toList());
	}
}
```
