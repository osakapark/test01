# entity 설정

## 01. Base Entity 설정
```
@MappedSuperclass
@EntityListeners(value = { AuditingEntityListener.class })
@Getter
abstract class BaseEntity {
    @CreatedDate
    @Column(name = "regdate", updatable = false)
    private LocalDateTime regDate;

    @LastModifiedDate
    @Column(name ="moddate")
    private LocalDateTime modDate;
}
```
1. @MappedSuperclass (교재 133page)
 해당 class 는 Table 로 생성하지 않음
 상속한 클래스에서생성

## 02. Child Entity 설정

Member Class
```java
public class Member extends BaseEntity {

    @Id
    private String email;

    private String password;

    private String name;
}
```

Board Class
```java
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@ToString(exclude = "writer")
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    private String title;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY) // 명시적으로 Lazy 로딩 지정	
    private Member writer;	//연관관계 지정
}
```
 1. @Id : PK 의미 
 2. @GeneratedValue (교재 52page)
    IDENTITY : MySQL, MariaDB 에서 auto increment
 3. @Builder : 객체 생성
 4. @ManyToOne
     N(Board): 1(Member) 관계 설정
 5. Lazy 설정시 사용하는 곳에서 @Transactional 필요 (교재 247Page)    
    - 필요할 때 다시 데이터베이스 연결
    - Lazy 로딩 사용하지 않으면 자동으로 join 처리 (교재 248page)
  6. ToString(exclude)
    - 연관관계가 있는 객체는, Lazy 로딩 사용시 exclude 하는 편이 좋음 (교재 248page)
  7. 지연로딩을 기본으로 사용하고, 상황에 맞게 필요한 방법을 찾는다. (교재 249page)

```java
@Transactional
@Test
public void testRead1() {
    Optional<Board> result = boardRepository.findById(100L);

    Board board = result.get();
    System.out.println(board);
    System.out.println(board.getWriter());
}
```    


Reply Class
```java
@ToString(exclude = "board")
public class Reply extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long rno;

    private String text;

    private String replyer;

    @ManyToOne
    private Board board; //연관관계 지정
}
```