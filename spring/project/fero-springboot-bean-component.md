# FERO-springboot-bean-component

### 문제상황

```shell
2025-02-11 21:36:11.912  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : =====================
2025-02-11 21:36:12.926  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : === WebSocket Status ===
2025-02-11 21:36:12.927  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : Sessions: {}
2025-02-11 21:36:12.927  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : =====================
2025-02-11 21:36:12.927  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : === WebSocket Status ===
2025-02-11 21:36:12.927  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : Sessions: {a8742809-205f-3d25-e5fb-d644585c9869=StandardWebSocketSession[id=a8742809-205f-3d25-e5fb-d644585c9869, uri=ws://localhost:8076/api/v1/videorooms]}
2025-02-11 21:36:12.927  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : =====================
2025-02-11 21:36:13.925  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : === WebSocket Status ===
2025-02-11 21:36:13.925  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : Sessions: {}
2025-02-11 21:36:13.925  INFO 11896 --- [pool-2-thread-1] com.ssafy.api.handler.SignalingHandler   : =====================
```

#### 원인

중복된 빈 등록으로 인해 서로 다른 인스턴스가 사용됨

* 같은 클래스를 @Bean으로 등록하고 @Component로도 등록하여 Spring 컨테이너에서 두 개의 서로 다른 인스턴스를 관리하게 됨.
* 특정 로직에서 하나의 빈 인스턴스에 데이터를 저장했지만, 다른 인스턴스를 참조하는 코드에서는 빈 맵처럼 보이게 되었음.

#### 해결책

* @Bean으로 등록하고 @Component로 등록한 것을 지웠음

### 추가 개념

#### springboot의 IoC

IoC는 객체의 생성과 관리 주체를 개발자가 아닌 프레임워크(Spring Container)가 담당하는 개념을 의미함.\
즉, 프로그램의 흐름을 개발자가 직접 제어하는 것이 아니라 프레임워크가 제어함.

1. DI (Dependency Injection, 의존성 주입)\
   IoC의 핵심 구현 방식으로, 객체의 의존성을 스프링이 자동으로 주입해주는 것.
2. 스프링 컨테이너 (Spring Container)\
   스프링 컨테이너는 객체의 생성, 관리, 소멸을 담당하는 IoC 컨테이너임.\
   애플리케이션 실행 시 자동으로 빈(Bean)을 생성하고 관리함.
3. 제어 흐름의 역전\
   기존 방식은 개발자가 객체를 생성하고 관리하지만,\
   IoC에서는 스프링이 객체를 생성하고 개발자는 가져다 쓰기만 하면 됨.

#### @Bean

스프링에서 \*\*빈(Bean)\*\*이란, 스프링 컨테이너가 관리하는 객체를 의미함.\
즉, new 키워드로 직접 생성하는 것이 아니라, 스프링이 자동으로 생성하고 관리해주는 객체임.

* @Configuration 클래스 내부에서 @Bean을 선언하면, Spring이 해당 메서드를 실행한 결과를 빈으로 등록함.
* 이 방식은 명시적으로 특정한 객체를 빈으로 등록할 때 사용함. (수동 등록)

#### @Component

* @Component 애너테이션이 붙은 클래스는 스프링 컨테이너에 의해 자동으로 관리됨. (자동 등록)
* 컴포넌트는 **빈(Bean)의 일종**이며, 스프링이 객체를 생성하고, 필요할 때 주입(Dependency Injection, DI) 해줌.

| 어노테이션       | 역할                     |
| ----------- | ---------------------- |
| @Component  | 일반적인 컴포넌트(빈)           |
| @Service    | 서비스 클래스 (비즈니스 로직 담당)   |
| @Repository | 데이터 접근 객체 (DAO, DB 관련) |
| @Controller | MVC 패턴에서 컨트롤러 역할       |
