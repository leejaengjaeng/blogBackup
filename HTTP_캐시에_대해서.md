# 캐시란 무엇일까요? 

단순히 생각해보면 어떤 요청에 대한 응답을 저장해두고 동일한 요청이 왔을때 저장된 값을 그대로 응답하는 방식으로 생각해볼수 있을 것 같습니다.

그렇다면 캐시는 어디에 쓰이고 있는걸까요?

Spring을 주로 사용해서 그런지 가장 먼저 생각나는건 `@Cacheable` 어노테이션을 통해서 함수의 결과를 캐싱하는게 떠오르네요.
하지만 조금만 더 생각해보면 캐시는 훨씬 광범위 하게 사용되고 있다는걸 알 수 있습니다.

RAM 자체도 하나의 캐시로 볼 수 있고, 뒤로가기와 같은 브라우저의 캐시, JPA의 영속성 컨텍스트도 캐시라고 볼 수 있겠네요.
이렇게 다양하게 캐시가 사용되고 있지만 이번 글에선 Http Cache에 대해서 작성해보려고 합니다.

# Http Cache란?

