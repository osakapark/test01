# Repository  설정

## 1. Repository Interface 선언
```
public interface MemberRepository extends JpaRepository<Member, String> {
}
```
1. Interface 를 선언하는 것 만으로, 모든 처리가 끝나는 마법(=자동으로 빈 생성) (교재 56page)
2. Entity Type 정보, @Id Type 설정

## 2. 관계 확인
```java
public interface BoardRepository extends JpaRepository<Board, Long> {
    //연관관계 있음
	@Query("select b, w from Board b left join b.writer w where b.bno =:bno")
	Object getBoardWithWriter(@Param("bno") Long bno);
    
    //연관관계 없음
    @Query("SELECT b, r FROM Board b LEFT JOIN Reply r ON r.board = b WHERE b.bno = :bno")
	List<Object[]> getBoardWithReply(@Param("bno") Long bno);

	@Query(value = "SELECT b, w, count(r) " + " FROM Board b " + " LEFT JOIN b.writer w "
			+ " LEFT JOIN Reply r ON r.board = b " + " GROUP BY b", countQuery = "SELECT count(b) FROM Board b")
	Page<Object[]> getBoardWithReplyCount(Pageable pageable);

	@Query("SELECT b, w, count(r) " + " FROM Board b LEFT JOIN b.writer w " + " LEFT OUTER JOIN Reply r ON r.board = b"
			+ " WHERE b.bno = :bno")
	Object getBoardByBno(@Param("bno") Long bno);
}
```

```java
public class Board extends BaseEntity {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long bno;

	private String title;

	private String content;

	@ManyToOne(fetch = FetchType.LAZY) // 명시적으로 Lazy 로딩 지정
	private Member writer; // 연관관계 지정
}
```

### 2.1 JPQL left outer join (연관관계 있는 경우)
```java
@Test
public void testReadWithWriter() {
    Object result = boardRepository.getBoardWithWriter(100L);
    Object[] arr = (Object[]) result;
    System.out.println("-------------------------------");
    System.out.println(Arrays.toString(arr));
}
```

1.내부 엔티티 사용시에는 left outer join 뒤에 on 불필요 (신기하군)
2.Lazy loadin 이지만, join 처리가 한번에 됨

### 2.2 JPQL left outer join (연관관계 없는 경우)
```java
@Test
public void testGetBoardWithReply() {
    List<Object[]> result = boardRepository.getBoardWithReply(100L);
    for (Object[] arr : result) {
        System.out.println(Arrays.toString(arr));
    }
}
```

## 3. 삭제
```java
public interface ReplyRepository extends JpaRepository<Reply, Long> {
    @Modifying
    @Query("delete from Reply r where r.board.bno =:bno ")
    void deleteByBno(Long bno);
}
```