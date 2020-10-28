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
}
```