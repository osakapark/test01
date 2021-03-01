# Repository  확장

## 1. 규칙
(교재 294page)
- 쿼리 메소드나 @Query 등으로 처리할 수 없는 기능은 별도의 인터페이스로 설계
- 별도의 인터페이스에 대한 구현 클래스를 작성. 이 때 QuerydslRepositorySupport 라는 클래스를 부모클래스로 사용
- 구현 클래스에 인터페이스의 기능을 Q도메인 클래스와 JPQLQuery 를 이용해서 구현


```java
public interface SearchBoardRepository {
	Board search1();

	Page<Object[]> searchPage(String type, String keyword, Pageable pageable);
}
```

```java
public class SearchBoardRepositoryImpl extends QuerydslRepositorySupport implements SearchBoardRepository {

	public SearchBoardRepositoryImpl() {
		super(Board.class);
	}

	@Override
	public Board search1() {

		log.info("search1........................");

		QBoard board = QBoard.board;
		QReply reply = QReply.reply;
		QMember member = QMember.member;

		JPQLQuery<Board> jpqlQuery = from(board);
		jpqlQuery.leftJoin(member).on(board.writer.eq(member));
		jpqlQuery.leftJoin(reply).on(reply.board.eq(board));

		JPQLQuery<Tuple> tuple = jpqlQuery.select(board, member.email, reply.count());
		tuple.groupBy(board);

		log.info("---------------------------");
		log.info(tuple);
		log.info("---------------------------");

		List<Tuple> result = tuple.fetch();

		log.info(result);

		return null;
	}

	@Override
	public Page<Object[]> searchPage(String type, String keyword, Pageable pageable) {

		log.info("searchPage.............................");

		QBoard board = QBoard.board;
		QReply reply = QReply.reply;
		QMember member = QMember.member;

		JPQLQuery<Board> jpqlQuery = from(board);
		jpqlQuery.leftJoin(member).on(board.writer.eq(member));
		jpqlQuery.leftJoin(reply).on(reply.board.eq(board));

		// SELECT b, w, count(r) FROM Board b
		// LEFT JOIN b.writer w LEFT JOIN Reply r ON r.board = b
		JPQLQuery<Tuple> tuple = jpqlQuery.select(board, member, reply.count());

		BooleanBuilder booleanBuilder = new BooleanBuilder();
		BooleanExpression expression = board.bno.gt(0L);

		booleanBuilder.and(expression);

		if (type != null) {
			String[] typeArr = type.split("");
			// 검색 조건을 작성하기
			BooleanBuilder conditionBuilder = new BooleanBuilder();

			for (String t : typeArr) {
				switch (t) {
				case "t":
					conditionBuilder.or(board.title.contains(keyword));
					break;
				case "w":
					conditionBuilder.or(member.email.contains(keyword));
					break;
				case "c":
					conditionBuilder.or(board.content.contains(keyword));
					break;
				}
			}
			booleanBuilder.and(conditionBuilder);
		}

		tuple.where(booleanBuilder);

		// order by
		Sort sort = pageable.getSort();

		// tuple.orderBy(board.bno.desc());

		sort.stream().forEach(order -> {
			Order direction = order.isAscending() ? Order.ASC : Order.DESC;
			String prop = order.getProperty();

			PathBuilder orderByExpression = new PathBuilder(Board.class, "board");
			tuple.orderBy(new OrderSpecifier(direction, orderByExpression.get(prop)));

		});
		tuple.groupBy(board);

		// page 처리
		tuple.offset(pageable.getOffset());
		tuple.limit(pageable.getPageSize());

		List<Tuple> result = tuple.fetch();

		log.info(result);

		long count = tuple.fetchCount();

		log.info("COUNT: " + count);

		return new PageImpl<Object[]>(result.stream().map(t -> t.toArray()).collect(Collectors.toList()), pageable,
				count);
	}
}
```

```java
public interface BoardRepository extends JpaRepository<Board, Long>, SearchBoardRepository {
	@Query("select b, w from Board b left join b.writer w where b.bno =:bno")
	Object getBoardWithWriter(@Param("bno") Long bno);

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