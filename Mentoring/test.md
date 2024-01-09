## BoardService의 createBoard Test Code

```java
public CommonIdResponse createBoard(Long memberId, CreateBoardRequest createBoard) {
        // 게시글 저장
        Board board = createBoard.toEntity(memberId);
        boardMapper.save(board);
        // 이미지 정보 저장
        if (createBoard.isImageExist()) {
            saveImageInfo(board.getBoardId(), createBoard.getImageInfo());
        }
        return new CommonIdResponse(board.getBoardId());
    }
```

- 위 메서드를 테스트하려면 우선 단언이 2개 나와야한다. 게시글이 잘 저장됐는지, 이미지가 잘 저장됐는지

- 위 메서드는 제어할 수 없는 외부 환경에 영향을 받는다. 사용자의 요청에 이미지 정보가 있는지 없는지에 테스트는 다른 결과를 반환한다.

