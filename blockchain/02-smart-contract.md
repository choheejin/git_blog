# smart contract

## Smart Contract란?

* 1990년대에 Nick Szabo가 소개한 개념
  * 디지털 형식으로 명시된 서약들의 집합
  * 결코 스마트하지 않은 단순 컴퓨터 프로그램.
  * 법적 맥락 없음
  * 다소 잘못된 용어임에도 불구하고 자리잡음
* 블록체인에서의 정의: 불변의 컴퓨터 프로그램
  * 불변: 한번 배포되면 변경 불가
  * 결정적으로 실행한 결과가 모두 같음
  * EVM 위에서 동작
  * 탈중앙화된 World Computer 동일한 상태를 유지
* Smart Contract를 작성하는 언어
  * Solidity
    * [Solidity 문법 연습하기](https://cryptozombies.io/ko)
  * LLL
  * Viper
  * Assembly

## smart contract 배포와 호출

Smart Contract Code

⬇️ 컴파일

EVM Bytecode, ABI in JSON

⬇️ 트랜잭션 생성

```java
{
	...
	from: deployer’s adderss,
	to: 0x,
	data: bytecode,
	...
}
```

⬇️ 서명

sending transaction

> ByteCode: 작성한 smart contract 코드의 컴파일 된 코드\
> ABI(Application Binary Interface): contract에 정의되어 있는 functional list\
> CA(Contract Address): smart contract를 배포하게 되면, 그 id로 쓰게 될 주소
