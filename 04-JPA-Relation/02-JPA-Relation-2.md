# JPA Relation #2

## 이론

* @OneToOne
  * Entity를 1:1 관계로 연결해주는 어노테이션
  * `optional=false`이면 `inner join`으로 쿼리가 생성됨
  * `optional=true`로 하면 `left outer join`으로 쿼리가 생성됨

## 실전코드

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
