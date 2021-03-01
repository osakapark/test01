#DTO Entity 변환

# 1. 소스 확인 하기.. (일단 외우자..)
PageResultDTO.java
```java
    public PageResultDTO(Page<EN> result, Function<EN,DTO> fn ){
        dtoList = result.stream().map(fn).collect(Collectors.toList());       
    }
```
BoardService.java
```java
public interface BoardService {
	Long register(BoardDTO dto);

	PageResultDTO<BoardDTO, Object[]> getList(PageRequestDTO pageRequestDTO);

	default Board dtoToEntity(BoardDTO dto) {
		Member member = Member.builder().email(dto.getWriterEmail()).build();

		Board board = Board.builder() //
				.bno(dto.getBno()) //
				.title(dto.getTitle()) //
				.content(dto.getContent()) //
				.writer(member) //
				.build();
		return board;
	}

	default BoardDTO entityToDTO(Board board, Member member, Long replyCount) {

		BoardDTO boardDTO = BoardDTO.builder() //
				.bno(board.getBno()) //
				.title(board.getTitle()) //
				.content(board.getContent()) //
				.regDate(board.getRegDate()) //
				.modDate(board.getModDate()) //
				.writerEmail(member.getEmail()) //
				.writerName(member.getName()) //
				.replyCount(replyCount.intValue()) // int로 처리하도록
				.build();
		return boardDTO;
	}
}
```
1. 내부적으로 Member 생성했음
2. interface/default 이해할 것!

BoardServiceImpl.java
```java
@Override
public PageResultDTO<BoardDTO, Object[]> getList(PageRequestDTO pageRequestDTO) {

    log.info(pageRequestDTO);

    Function<Object[], BoardDTO> fn = (en -> entityToDTO((Board) en[0], (Member) en[1], (Long) en[2]));
    Page<Object[]> result = repository
            .getBoardWithReplyCount(pageRequestDTO.getPageable(Sort.by("bno").descending()));
    return new PageResultDTO<>(result, fn);
}
```

```java
@Test
public void testList() {

    // 1페이지 10개
    PageRequestDTO pageRequestDTO = new PageRequestDTO();

    PageResultDTO<BoardDTO, Object[]> result = boardService.getList(pageRequestDTO);

    for (BoardDTO boardDTO : result.getDtoList()) {
        System.out.println(boardDTO);
    }
}
```
#2 getBno() 지연로딩? 277 Page, 60Page 참고 하자

  


    
    
    public void insertMembers() {

        IntStream.rangeClosed(1, 100).forEach(i -> {

            Member member = Member.builder()
                    .email("user" + i + "@aaa.com")
                    .password("1111")
                    .name("USER" + i)
                    .build();

            memberRepository.save(member);
        });
    }

254 page
```java
@Test
public void testWithReplyCount() {
    Pageable pageable = PageRequest.of(0, 10, Sort.by("bno").descending());
    Page<Object[]> result = boardRepository.getBoardWithReplyCount(pageable);
    result.get().forEach(row -> {
        Object[] arr = (Object[]) row;
        System.out.println(Arrays.toString(arr));
    });
}
```    


