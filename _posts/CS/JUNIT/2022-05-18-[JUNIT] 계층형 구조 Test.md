---
title : "[JUNIT] 계층형 구조 Test"
tags : 
 - Junit
---



## 계층형 구조 Test

아프리카TV 검색팀의 동아줄 혁이 계층형 구조 테스트를 알려줘서 한번 봤는데 가독성이 매우 좋아 기록한다.

<br/>

#### Describe - Context - It

BDD(Behavior Driven Development) 패턴으로 코드의 행동을 설명하는 테스트 코드를 작성한다. TDD와 다른점이 있다고 하는데 사실 잘 이해가 되진 않는다. 최종 서비스의 동작에 중점을 두고 Test를 작성하는것으로 이해했다. given - when - then으로 작성하는것인데 참고한 블로그는 describe - context - it를 사용한다고 한다.

| 키워드   | 설명                                             |
| -------- | ------------------------------------------------ |
| Describe | 테스트 대상이 되는 클래스, 메소드를 명시한다.    |
| Context  | 테스트 메소드에 입력할 파라미터를 설명한다.      |
| It       | 테스트 대상 메소드가 무엇을 리턴하는지 설명한다. |

`Context`문을 작성할 때에는 반드시 `with`또는`when`으로 시작하도록 한다.

`It`문은 `ItReturnTrue`, `ItResponse400` 심플하게 설명한다.

<br/>

#### Test

```java
@WebMvcTest
@AutoConfigureDataJpa
@DisplayName("계층형 구조 Test")
public class JunitTest {

    @Autowired
    private MockMvc mockMvc;

    @BeforeAll
    static void setUp() {
        System.setProperty("runtimeEnv", "local");
    }

    public ResultActions getPost() throws Exception {
        ResultActions perform = mockMvc.perform(MockMvcRequestBuilders.get("/posts"));
        return perform;
    }

    @Nested
    @DisplayName("PostController 클래스 테스트")
    class PostControllerTest {

        @Nested
        @DisplayName("getPost 메소드는")
        class DescribeGetPost {

            @Nested
            @DisplayName("pageable 파라미터가 있다면")
            class ContextWithPageable {

                @Test
                @DisplayName("PagedModel을 리턴한다")
                public void ItReturnPagedModel() {
                    System.out.println("PagedModel");
                }
            }

            @Nested
            @DisplayName("pageable 파라미터가 없다면")
            class ContextWhenNotPageable {

                @Test
                @DisplayName("400으로 응답한다")
                public void ItResponse400() {
                    System.out.println("400 Response");
                }
            }
        }
    }
}
```

<br/>

#### Test Result

![image](https://user-images.githubusercontent.com/46040824/168967222-fe3b3edd-5554-4e7a-8d43-d4fdb7908242.png)

<br/>

#### Subject 메소드

어떤 메소드에 대해 Test를 한다고 가정할때 그 메소드를 호출하는것을 따로 함수로 빼두고 메소드에 대한 파라미터가 변경될때 빼두었던 Subject메소드만 수정하여 테스트 수정을 최소화 하는 것이다.

<br/>

###### 참고 : https://johngrib.github.io/wiki/junit5-nested/