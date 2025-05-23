# Servlet

## Servlet이란

* 동적 웹 페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술이다.
* Java 기반의 웹 어플리케이션에서 Http 요청을 처리하고 응답을 생성하는 서버 사이드 컴포넌트이다.

## 특징

* HTTP 요청 및 응답 처리
* 멀티쓰레드 지원
* 라이프사이클 자동 관리
* JSP와 연동 가능
* Java EE 표준 기술

## 동작과정

1. 클라이언트가 웹 브라우저에게 요청을 보냄
2. 웹 컨테이너(톰캣..)가 요청을 서블릿으로 전달
3. 서블릿이 요청을 처리하고 응답을 생성 (HttpServletRequest, HttpServletResponse)
4. 웹 컨테이너가 응답을 클라이언트에게 전송

## 생명주기

1. Constructor
   * Servlet 객체 생성할때 한번 오출
2. init()
   * 서블릿이 메모리에 로드될 때 한번 호출
   * 코드 수정으로 인해 다시 로드되면 다시 호출
3. service(), doGet(), doPost()
   * doGet(): Get 방식으로 data 전송시 호출
   * doPost(): Post 방식으로 data 전송시 호출
   * service(): 모든 요청은 service()를 통해서 doXXX() 메서드로 이동
4. destory()
   1. 서블릿이 메모리에서 해제되면 호출
   2. 코드가 수정되면 호출

## JSP vs Servlet

* JSP
  * HTML 문서 안에 JAVA 코드를 포함하고 있다
* Servlet
  * 자바 코드 안에 HTML을 포함하고 있다.
