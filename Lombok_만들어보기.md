# 삽질의 시작..

최근에 담당 서비스가 변경되고 팀분들의 배려로 적응의 시간을 갖고있습니다.  
덕분에 상대적으로 다른 분들에 비해서 바쁘지 않은 생활을 하고 있는데요

이 서비스에서 도움이 될 일이 뭐가 있을까 생각하다보니 기존에 찾아두신 개선 포인트들을 하나씩 처리해볼까 싶은 생각이 들었습니다.

![manyIssues](https://blog.kakaocdn.net/dn/bHhLlk/btq4boj8Hf8/pxRDEPBnlja5msHhS1mP0K/img.png)

처리할만한 이슈를 찾다가 아래의 두개 이슈를 처리하기로 결심했습니다.

![issues](https://blog.kakaocdn.net/dn/t8yAT/btq4cFrFOCp/9wXKttOCk5DvbGi6CWjKPk/img.png)

#874번 이슈는 lombok을 사용하고 있기때문에 코드마다 `@Slf4j` 어노테이션만 붙여주고,

생성되는 `log` 변수를 사용하여 로깅을 하도록 바꿔주면 되는 단순한 작업이었고

#1000번 이슈는 유틸성 클래스를 하나 만든 뒤 로깅을 `LogUtil.info(log, "메서드명", "파라미터");`와 같은 형태로 해주면 되는 마찬가지로 기술적인 내용보다는 시간만 걸리는 작업이었습니다.

단순히 ctrl+c, ctrl+v로 반복하면 되는 작업이긴 하지만.. 그런식으로 작업하기엔 보람도 없고 재미도 없기때문에 삽질을 시작하게 되었는데요.

lombok에서 `@Slf4j` 어노테이션을 통해 `log`란 변수를 코드에 생성해주는것 처럼

**제가 만든 `@PayLogging` 어노테이션을 통해, 로깅 유틸 자체를 `log`란 변수로 코드에 생성** 해주고 기존 `log.info`와 메서드 시그니쳐가 같은 메서드를 만들어서 단순 작업양을 줄이면서 추가로 필요한 메서드들을 더 구현하는 방식이었습니다.

---

# 하고싶은것

![slf4j](https://i.imgur.com/JtBjXpi.png)

  
위와 같은 기능을 구현하려면 lombok의 `@Slf4j`가 어떻게 코드를 생성해주는지에 대해서 알아야했습니다.

> [https://stackoverflow.com/questions/6107197/how-does-lombok-work](https://stackoverflow.com/questions/6107197/how-does-lombok-work)  
> [https://stackoverflow.com/questions/40400648/how-lombok-generates-code-onto-existing-class](https://stackoverflow.com/questions/40400648/how-lombok-generates-code-onto-existing-class)

요약하자면 lombok의 어노테이션들에 대해 동작하는 Annotation processor를 사용해서  
compile time에 AST(Abstract Syntax Tree)를 조작해 소스코드를 추가로 붙여주고 이후에 컴파일이 진행되는 식으로 동작한다고 합니다.

#### AST? (Abstract Syntax Tree)

> [https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81\_%EA%B5%AC%EB%AC%B8\_%ED%8A%B8%EB%A6%AC](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%EA%B5%AC%EB%AC%B8_%ED%8A%B8%EB%A6%AC)

간단히 보자면 컴파일러에서 코드의 구조를 표현하는데 사용하는 자료구조로 아래와 같은 형태로 이해할 수 있습니다.

![java-ast](https://i.imgur.com/siHo7Xy.png)

실제 lombok의 구현체를 보게되면 `com.sun.tools.javac` 패키지에 있는 AST를 다루기 위한 모델들 + 자체 제작한 클래스를 사용해서 소스코드를 생성하고 있는걸 볼 수 있습니다.

![lombok코드](https://i.imgur.com/EpjcaKC.png)

> 직접 타이핑 하면 `private static final Logger log = LoggerFactory.getLogger(Example.class)` 한줄이지만 소스코드를 작성하기 위한 코드는 꽤나 기네요..

이제 방법을 알았으니 뚝딱 만들면 되겠군!  
하는 터무니 없는 생각을 하자 마자 여러가지 문제가 발생했습니다..

---

# 문제점

#### 1\. 어떻게 등록하지..?

lombok의 코드를 참고해서 **어노테이션 프로세서** 를 하나 만들고 보니  
단순히 소스코드에 추가한다고 동작하는게 아니라는 사실을 알게되었습니다.

-   검색해보니 별도의 jar 파일로 패킹해서 javac 옵션에 넣어줘야 한다는것을 확인했습니다.

#### 2\. jar 패킹, javac 옵션..?

별도로 패킹해서 실행 옵션에 넣어줘야 한다는점은 더이상 이걸 하고있을 필요가 없다는 생각을 들게했습니다.

관리 포인트가 늘어나고, 실행 스크립트를 다 바꿔줘야하고, 시간이지나서 이런식으로 동작한다는걸 기억하는 사람이 없어진다면 서비스에 재앙이 닥쳐올게 분명했습니다.

-   다행히 빌드도구로 해결이 가능한 문제였습니다.  
    메이븐 플러그인을 통해 사용할 **어노테이션 프로세서** 를 설정해줄 수 있었습니다.

#### 3\. 메이븐에 설정해 줬는데 왜 안돼..?

스택오버플로우의 힘을 빌어 드디어 메이븐 세팅도 완료했습니다.  
하지만 여전히 두가지 문제가 있었습니다.

-   첫번째는 **어노테이션 프로세서** 를 별도의 프로젝트로 두지않고 pom.xml을 기존것과 공유하다 보니, javac 실행시점에 제가 만든 **어노테이션 프로세서** 가 컴파일 되지 않은 상태로 존재하기 때문에 찾지 못하는 문제였습니다.
    -   이 문제는 메이븐의 lifecycle을 이용해서 **어노테이션 프로세서** 만 우선 컴파일해서 클래스 패스에 넣어주고,  
        나머지 코드를 이후에 다시 한번 컴파일 하면서 플러그인을 통해 만들어둔 **어노테이션 프로세서** 를 찾을 수 있도록 설정을 변경해서 해결 할 수 있었습니다.  
        
        ![pom-xml](https://i.imgur.com/mDmTKSn.png)
        
-   두번째는 **어노테이션 프로세서** 에 동작 확인을 위해 찍어둔 로그는 확인할 수 있지만 IDE 상에서 디버깅이 안되는 문제가 있었습니다.
    -   이건 생각해보면 당연한 문제였는데 평소에 IDE에 잡아둔 디버깅은 **배포한 파일**에 대한 런타임 디버깅 세션이었고, 제가 하고 싶은것은 **maven** 의 컴파일 타임 디버깅 세션이 필요한 것이었습니다.  
        결론적으로 IDE에 remote 디버깅 설정을 하고 `mvnDebug` 명령어를 통해서 메이븐 디버깅 세션에 연결해주는것으로 해결할 수 있었습니다.

#### 4\. 이거 못하는건가..?

문제되던 상황은 다 해결했고 연동 테스트를 위해 만들어뒀던 **어노테이션 프로세서** 를 가지고 **`@PayLogging` 어노테이션을 통해, `private static final LogUtil log = LogUtil.instance();`** 한줄을 생성해주는 재밌는 작업만 남았습니다.

생각보다 레퍼런스가 더 없어서 약간 고생을 했지만 마찬가지로 lombok 코드를 분석하면서 `private static final LogUtil log = LogUtil.instance();`를 나타내는 `JCVariableDecl`을 만들었고 AST상에서 추가되어야 하는 위치까지 뽑아낼 수 있었습니다.

이제 트리에 put만 해주면 끝인데... 아무리봐도 관련된 함수를 찾을 수 없었습니다.

-   알고보니 이부분이 lombok이 해킹이라고 취급되는 부분이었습니다.  
    어노테이션 프로세서는 기본적으로 현재 컴파일중인 파일을 변경하지 못하고 컴파일시 추가할 새로운 파일(.java)의 생성만이 가능하다고 합니다.
-   다행히 lombok 코드를 디버깅해서 AST를 직접 수정해주는것으로 보이는 코드를 찾을 수 있었고, 유사한 형태로 제 코드에 적용해보니 정상적으로 코드가 추가된 것을 확인 할 수 있었습니다.

---

# 현재 상황

![KakaoTalk_20200809_044937483](https://i.imgur.com/lQFCP9r.png)

![KakaoTalk_20200809_044937550](https://i.imgur.com/9ZUaRvZ.png)

> **.java** 파일에는 없는 `log` 변수가 **.class** 파일에는 들어있는것을 볼 수 있습니다.  
> `PayLoggingUtil`에 추가해놓은 `hello()` 메서드도 정상적으로 동작합니다.

처음 의도대로 `@PayLogging` 어노테이션을 통해 `PayLoggingUtil`을 `log` 변수로 추가해주는것에 성공했습니다.

하지만 IDE 상에서 여전히 `log` 변수를 찾지 못해 빨간줄로 표시되는 이슈와(정상적으로 컴파일 됩니다 ㅎㅎ..) `PayLoggingUtil`의 기능 구현이 남았습니다.

---

# 결론

목적은 달성했지만 이걸 사용하는게 맞을까에 대한 의문이 아직 남아있습니다.  
제가 생각하는 장점과 단점은 아래와 같습니다.

**Pros.**

-   기존과 유사한 형태로 자유롭게 확장 가능한 logger를 사용할 수 있습니다.
-   흔히 사용하지 않는 기술을 사용해 볼 수 있습니다.

**Cons.**

-   이후에 **어노테이션 프로세서** 에 대한 수정이 필요하게 되면 레퍼런스가 많지 않아서 까다로울 수 있습니다.
-   JAVA 버전이 9(모듈화) 이상으로 올라가면 어느정도 수정이 필요할 수 있습니다.
-   위의 이유들로 긁어부스럼을 만드는것 일 수 있습니다.

제 결론은.. 팀분들이 원하시면 넣고 아니면 새로 알게된 내용만 공유드리고 만족하기로 했는데요 

굳이 긁어부스럼을 만들어서 뭐하나 싶은 마음이 큰것 같습니다.

서비스중인 코드에 추가되진 못했지만 개인적으로는 재미있는 내용이었습니다 :)
