# SpringFramework

## 등장배경

* EJB, 현실에서의 반영이 어렵다
  * 코드 수정 후 반영하는 과정 자체가 거창해 기능은 좋지만 복잡한 스펙으로 인한 개발의 효율성이 떨어짐
  * 어플리케이션 테스트 하기 위해서는 반드시 EJB 서버가 필요하다
* 웹 사이트가 점점 커지면서 엔터프라이즈 급의 서비스가 필요하게 됨
  * Enterprise System: 서버에서 동작하며 기업의 업무를 처리해주는 시스템을 말함
  * 세션빈에서 Transaction 관리가 용이함
  * 로깅, 분산처리, 보안등
* 자바진영에서는 EJB가 엔터프라이즈급 서비스로 각광을 받게 됨
  * EJB 스펙에서 정의된 인터페이스에 따라 코드를 작성하므로 기존에 작성된 POJO를 변경해야 함
  * 컨테이너에 배포를 해야 테스트가 가능해 개발속도가 저하됨
  * 배우기 어렵고, 설정해야하는 부분이 많음
  * EJB는 RMI를 기반으로 해야하는 서버이므로 무거운 컨테이너다
* Rod Johnson이 ‘Expert One-on-One J2EE Development without EJB’라는 저서에서 EJB를 사용하지 않고 엔터프라이즈 어플리케이션을 개발하는 방법을 소개함
  * AOP나 DI 같은 새로운 프로그래밍 방법론으로 가능
  * POJO로 선언적 프로그래밍 모델이 가능해짐
* 점차 POJO + 경량 프레임워크를 사용하기 시작
  * POJO (Plain Old Java Object)
    * 특정 프레임워크나 기술에 의존적이지 않은 자바 객체
    * 특정 기술에 종속적이지 않기 때문에 생산성, 이식성 향상
    * Plain: component interface를 상속받지 않는 특징
    * Old: EJB 이전의 java class를 의미
  * 경량 프레임워크
    * EJB가 제공하는 서비스를 지원해줄 수 있는 프레임워크 등장
    * Hibernate, JDO, myBatis, Spring

### 결국 스프링의 핵심가치는…

엔터프라이즈급 서비스 기술의 혼란속에서 잃어버린 객체지향 기술을 되찾자!

## SpringFramework란?

* 엔터프라이즈 급 애플리케이션을 만들기 위한 모든 기능을 종합적으로 제공하는 경량화된 솔루션이다.
  * 개발자가 복잡하고 실수하기 쉬운 Low Level에 신경쓰지 않고 Business Logic 개발에 전념할 수 있도록 해준다.
* Java EE가 제공하는 다수의 기능을 지원하고 있기 때문에, Java EE를 대체하는 Framework로 자리잡고 있다.
* SpringFramework는 Java EE가 제공하는 다양한 기능을 제공하는 것 뿐만 아니라, DI(Dependency Injection)나, AOP(Aspect Oriented Programming)과 같은 기능도 지원한다.

## SpringFramework의 구조

* Spring 삼각형
  * Enterprise Application 개발시 복잡함을 해결하는 Spring의 핵심
    1. POJO (Plain Old Java Object)
       1. 특정환경이나 기술에 종속적이지 않은 객체지향 원리에 충실한 자바 객체
       2. 테스트하기 용이하며, 객체지향 설계를 자유롭게 적용할 수 있다.
    2. PSA (Portable Service Abstraction)
       1. 환경과 세부기술의 변경과 관계없이 일관된 방식으로 기술에 접근할 수 있게 해주는 설계 원칙
       2. 트랜잭션 추상화, OXM 추상화, 데이터 액세스의 Exception 변환 기능 등 기술적인 복잡함은 추상화를 통해 Low Level의 기술 구현 부분과 기술을 사용하는 인터페이스로 분리
       3. 예를 들어, 데이터베이스에 관계없이 동일하게 적용할 수 있는 트랜잭션 처리방식
    3. IoC/DI (Dependency Injection)
       1. DI는 유연하게 확장 가능한 객체를 만들어두고 객체간의 의존관계는 **외부**에서 다이나믹하게 설정
    4. AOP (Aspect Oriented Programming)
       1. 관심사의 분리를 통해서 소프트웨어의 모듈성을 향상
       2. 공통 모듈을 여러 코드에 쉽게 적용 가능

## IoC(Inversion of Control, 제어의 역행)

* 객체지향 언어에서 Object 간의 연결관계를 런타임에 결정
* 객체간의 관계가 느슨하게 연결됨(loosely-coupling)
* IoC의 구현 방법 중 하나가 DI(Dependency Injection)

### IoC 유형

1. Dependency Lookup
   1. JNDI Lookup
   2. 컨테이너가 lookup context를 통해서 필요한 Resource나 Object를 얻는 방식
   3. JNDI 이외의 방법을 사용한다면, JNDI 관련 코드를 오브젝트 내에서 일일이 변경해주어야 함
   4. Lookup한 Object를 필요한 타입으로 Casting 해주어야 함
   5. Naming Exception을 처리하기 위한 로직이 필요
2. Dependency Injection
   1. Setter Injection
   2. Constructor Injection
   3. Method Injection
   4. Object에 Lookup 코드를 사용하지 않고 컨테이너가 직접 의존 구조를 Object에 설정할 수 있도록 지정해주는 방식
   5. Object가 컨테이너의 존재여부를 알 필요가 없음
   6. Lookup 관련된 코드들이 Object 내에서 사라짐

💡

Constructor Injection을 더 권장하는 이유에 대해서 찾아보자

### Container

* 개념
  * 객체의 생성, 사용, 소멸에 해당하는 라이프사이클을 담당한다.
  * 라이프사이클을 기본으로 애플리케이션 사용에 필요한 주요 기능을 제공
* 기능
  * 라이프사이클 관리
  * Dependency 객체 제공
  * Thread 관리
  * 기타 애플리케이션 실행에 필요한 환경
* 필요성
  * 비즈니스 로직 외에 부가적인 기능들에 대해서는 독립적으로 관리되기 위함
  * 서비스 lookup 이나 Configuration에 대한 일관성을 갖기 위함
  * 서비스 객체를 사용하기 위해 각각 **Factory 또는 Singleton 패턴을 직접 구현하지 않아도 됨** ⇒ Bean Factory가 스프링 내부적으로 제공됨
* IoC Container
  * 오브젝트의 생성과 관계 설정, 사용, 제거 등의 작업을 애플리케이션 코드 대신 독립된 컨테이너가 담당
  * 컨테이너가 코드 대신 오브젝트에 대한 제어권을 가지고 있어 IoC라고 부름
  * 이런 이유로, 스프링 컨테이너를 IoC 컨테이너라고 부르기도 함
  * 스프링에서 IoC를 담당하는 컨테이너에는 **BeanFactory, ApplicationContext**가 있음
* Spring DI Container
  * Spring DI Contianer가 관리하는 객체를 Bean이라고 하고, 이 빈들의 생명주기를 관리하는 의미로 빈팩토리라고 함.
  * BeanFactory에 여러가지 컨테이너 기능을 추가하여 ApplicationContext라고 함
  * BeanFactory
    * Bean을 등록, 생성, 조회, 반환 관리
    * 일반적으로 BeanFactory보다는 이를 확장한 ApplicationContext를 사용
    * getBean() method 정의되어있음
  * ApplicationContext
    * Bean을 등록, 생성, 조회, 반환 관리기능은 BeanFactory와 동일
    * Spring의 각종 부가 서비스를 추가로 제공
    * Spring이 제공하는 ApplicationContext 구현클래스는 여러가지 종류가 존재

### AOP(Aspect Oriented Programming)

* 핵심 관심 사항과 공통(부가) 관심 사항
  * 핵심 관심 사항(core concern)과 공통 관심 사항(cross-cutting concern)
    * ex) 보안, 로그
  * 기존 OOP에서는 공통관심사항을 여러 모듈에서 적용하는데 있어 중복된 코드를 양상하는 한계가 존재함
  * 이를 해결하기 위해 AOP가 등장
  * Aspect Oriented Programming은 문제를 해결하기 위한 핵심 관심 사항과 전체에 적용되는 공통 관심 사항을 기준으로 프로그래밍함으로써 공통 모듈을 손쉽게 적용할 수 있게 함
* 분리한 부가 기능을 Aspect라는 독특한 모듈 형태로 만들어서 설계하고 개발하는 방법
* OOP를 적용하여도 핵심기능에서 부가기능을 쉽게 분리된 모듈로 작성하기 어려운 문제점을 AOP가 해결
* AOP는 부가기능을 Aspect로 정의하여, 핵심 기능에서 부가기능을 분리함으로써 핵심 기능을 설계하고 구현할 때 객체지향적인 가치를 지킬 수 있도록 도와주는 개념

### AOP 적용 예

* 간단한 메소드의 성능 검사
  * 개발 도중 특히 DB에 대량의 데이터를 넣고 빼는 등의 배치 작업에 대하여 시간을 측정해보고 쿼리를 개선하는 작업은 매우 의미가 있다. 이 경우, 매번 해당 메소드의 처음과 끝에 System.currentTimeMillis();를 사용하거나, 스프링이 ㅈ제공하는 StopWatch 코드를 사용하기는 번거로움
  * 이런 경우, 해당 작업을 하는 코드를 밖에서 설정하고 해당 부분을 사용하는 것이 편리함
* 트랜잭션 관리
  * 트랜잭션의 경우, 비즈니스 로직의 전후에 설정
  * 하지만, 매번 사용하는 트랜잭션이 Try-catch의 코드는 번거롭고, 소스를 더욱 복잡하게 보여줌

> Thread Pool에 대해서 더 알아보자
