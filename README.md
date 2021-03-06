실전! Querydsl
==============

목차
----

[1. Querydsl 소개](#1_querydsl-소개)

[2. 프로젝트 환경설정](#2_프로젝트-환경설정)

[3. 예제 도메인 모델](#3_예제-도메인-모델)

[4. 기본 문법](#4_기본-문법)

[5. 중급 문법](#5_중급-문법)

[6. 실무활용 순수 JPA와 Querydsl](#6_실무활용-순수-jpa와-querydsl)

[7. 실무활용 스프링 데이터 JPA와 Querydsl](#7_실무활용-스프링-데이터-jpa와-querydsl)

[8. 스프링 데이터 JPA가 제공하는 Querydsl 기능](#8_스프링-데이터-jpa가-제공하는-querydsl-기능)

---

### **1_Querydsl 소개**

#### 1.1 소개

-	실무에서는 복잡한 쿼리, 동적 쿼리 작성할 일이 많음

	-	이러한 문제를 해결할 수 있는 방법 Querydsl
	-	최신 자바 백엔드 기술의 마지막 퍼즐 조각

-	Querydsl

	-	쿼리를 자바 코드로 작성
	-	문법 오류를 컴파일 시점에 잡아줌
	-	동적 쿼리 문제 해결
	-	쉬운 SQL 스타일 문법
	-	**실무 필수 기술**

---

### **2_프로젝트 환경설정**

#### 2.1 프로젝트 생성

-	기존과 동일하게 https://start.spring.io/ 에서 생성

-	테스트 코드 실행 시 gradle 이 시작 될 때 (최신 인텔리제이에서..)

	-	build, execution, deployment
		-	Run Tests using : 인텔리제이로 변경

#### 2.2 Querydsl 설정과 검증

-	querydsl 은 큐타입을 뽑아내야함

-	gradle 설정 추가 후 큐타입 확인

#### 2.3 ~ 2.4 라이브러리 살펴보기, H2 데이터베이스 설치

-	querydsl-apt
	-	코드 generation 용도
	-	큐파일  
-	querydsl-jpa (다양한 쿼리들을 지원하고 jpa용임)
	-	애플리케이션에서 사용하는 코드 짤때 필요한 라이브러리
	-	selectFrom 등

#### 2.5 스프링 부트 설정 - JPA, DB

-	테스트 코드에 @Transactional 이 있으면 롤백함. @Commit 어노테이션을 달아주자

---

### **3_예제 도메인 모델**

#### 3.1 예제 도메인 모델과 동작 확인

-	예제 도메인 생성함
	-	Member
	-	Team

---

### **4_기본 문법**

#### 4.1 시작 - JPQL vs Querydsl

-	queryFactory를 만들때 엔티티매니저를 넘겨줘야함

-	querydsl은 기본적으로 자동으로 prepared statement 의 파라미터 바인딩 방식 사용

#### 4.2 기본 Q-Type 활용

-	static import를 통해 깔끔하게 사용하자

-	실행되는 JPQL 확인하기 위한 옵션

	-	spring.jpa.properties.hibernate.use_sql_comments: true

-	같은테이블을 조인해서 사용해야할 경우 QMember m1 = new QMember("m1"); 와 같이 사용할 것

	-	그렇지 않으면 거의 사용할 일 없음

#### 4.3 검색 조건 쿼리

-	jpql에서 지원하는 조건 다 있음

-	메서드 체인으로 연결 가능

#### 4.4 결과 조회

-	fetch() : 리스트 조회, 데이터 없으면 빈 데이터 반환

-	fetchOne() : 단 건 조회

	-	결과가 없으면 null
	-	결과가 둘이 상이면 NonUniqueResultRexception

-	fetchFirst() : limit(1).fetchOen();

-	fetchResults() : 페이징 정보 포함, , total count 쿼리 추가 실행

	-	페이징이 복잡해지면 쓰지말 것 (쿼리 2개를 나눠서 사용 권장)

-	fetchCount() : count 쿼리로 변경해서 count 수 조회

#### 4.5 ~ 4.7 정렬, 페이징, 정렬

-	집합 tuple

	-	실무에선 많이 쓰지않
	-	dto로 직접 뽑아서 사용함

#### 4.8 조인 - 기본 조인

-	조인의 기본 문법은 첫번 째 파라미터에 조인 대상을 지정하고, 두 번쨰 파라미터에 별칭으로 사용할 Q타입을 지정하면 된다.

-	from 절에 여러 엔티티를 선택해서 세타조인 가능 (연관관계없는 필드 조인)

	-	외부조인이 불가능함
	-	join on 을 사용하면 외부 조인 가능함!

#### 4.9 조인 - on 절

-	on절을 활용한 조인(JPA 2.1부터 지원)

	-	조인 대상 필터링
	-	연관관계 없는 엔티티 외부 조인

-	on절을 활용해 조인 대상을 필터링 할 때 외부조인이 아니라 내부조인을 사용하면, where절에서 필터링하는 것과 기능이 동일하다.

	-	따라서 on절을 활용한 조인 대상 필터링을 사용할 때, **내부조인이면 익숙한 where 절로 해결하고**, 정말 외부 조인이 필요한 경우에만 이 기능을 활용하자

-	하이버네이트 5.1 부터 on을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가됨 (내부조인도 가능)

-	주의 문법을 잘봐야함. leftJoin() 부분에 일반 조인과 다르게 엔티티 하나만 들어감

	-	일반 조인 : leftJoin(member.team, team)
	-	on 조인 : from(member).leftJoin(team).on(xx)

#### 4.10 조인 - 페치 조인

-	페치 조인은 SQL 에서 제공하는 기능은 아니다. SQL 조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능이다. (주로 성능 최적화에서 사용함)

#### 4.11 서브 쿼리

-	com.querydsl.jpa.JPAExpressions 사용

-	static import 가능

-	from 절의 서브쿼리 한계

	-	JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인뷰)는 지원하지 않는다.
	-	당연히 Querydsl도 지원하지 않는다.
	-	하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다.
	-	Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

-	from 절의 서브쿼리 해결방안

	-	서브쿼리를 join으로 변경한다. (불가능한 상황도 있음...)
	-	애플르케이션에서 쿼리를 2번으로 불리해서 실행한다.
	-	native SQL을 사용한다.

-	디비는 데이터만 필터링하고 그룹핑하고 가져오고, 로직들은 애플리케이션에서 끝내야한다. (서브쿼리 줄일 수 있다...)

#### 4.12 Case 문

-	select, where 에서 사용 가능

-	써야하는 지에 대해선 고민해야함

	-	**권장하는 건 DB에선 가급적인 이런 문제를 해결하지 말 것!**
	-	애플리케이션에서 해결

#### 4.13 상수, 문자 더하기

-	상수가 필요하면 Expressions.constant(xxx) 사용

-	문자가 아닌 타입들은 stringValue()를 통해 문자로 변환할 수 있음 (ENUM 처리할 때도 많이 사용됨)

---

### **5_중급 문법**

#### 5.1 프로젝션의 결과 반환 - 기본

-	프로젝션 : select 대상 지정

	-	프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있음
	-	프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회

-	튜플 조회

	-	프로젝션이 둘 이상일 때 사용

-	튜플은 리파티토리 계층안에서 정도만 사용할 것

	-	밖에서 사용할 땐 DTO 로 변환해서 사용할 것 (튜플이 querydsl 종속적인 타입이기 때문)

#### 5.2 프로젝션과 결과 반환 - DTO 조회

-	순수 JPA에서 DTO 조회 방법

	-	new 명령어를 사용해야함
	-	dto 패키지 이름을 다 적어줘야해서 지저분함
	-	생성자 방식만 지원함

-	querydsl에서 dt 조회는 3가지 방법 지원

	-	프로퍼티 접근 (setter)
	-	필드 직접 접근 (바로 필드에 들어감. setter 없어도됨)
	-	생성자 사용

-	프로퍼티나 필드 접근 생성 방식에서 이름이 다를 때 해결 방안

	-	ExpressionUtils.as(source, alias) : 필드나 서브 쿼리에 별칭 적용
	-	username.as("memberName") : 필드에 별칭 적용

#### 5.3 프로젝션과 결과 반환 - @QueryProjection

-	dto가 Q타입으로 생김

	-	컴파일 시점에 타입체크 가능

-	단점

	-	q파일 생성 필요
	-	querdsl 에 대한 의존성을 가지게 됨

#### 5.4 동적쿼리 - BooleanBuilder 사용

-	동적쿼리를 해결하는 두가지 방식

	-	BooleanBuilder
	-	Where 다중 파라미터 사용

#### 5.5 동적쿼리 - Where 다중 파라미터 사용

-	실무에서 좋아하는 방법임

-	where 조건에 null 값은 무시된다.

-	메서드로 동적쿼리 해결가능

	-	조립도 가능
	-	재활용 가능
	-	쿼리 자체의 가독성이 높아짐

-	null 체크는 주의해서 사용할 것

#### 5.6 수정, 삭제 벌크 연산

-	바로 DB의 값을 바꿔버리기 때문에 주의

-	벌크 연산 후에는 영속성 컨텍스트를 초기화해야 한다.

#### 5.7 SQL function 호출하기

-	SQL function은 JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.

-	lower 같은 ansi 표준 함수들은 querydsl 상당부분 내장하고 있음

---

### **6_실무활용 순수 JPA와 Querydsl**

#### 6.1 순수 JPA 리포지토리와 Querydsl

-	JPAQueryFactory 는 스프링 빈으로 등록해서 주입받아도 됨
	-	동시성 문제는 걱정안해도 된다고 함.
	-	스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저임
	-	가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.

#### 6.2 동적 쿼리와 성능 최적화 조회 - Builder 사용, Where 절 파라미터 사용

-	5장에서 설명했던 동적쿼리 작성을 활용함

#### 6.3 조회 API 컨트롤러 개발

-	샘플데이터 추가 하기

	-	테스트에 영향이 없도록 프로파일을 설정하자

	-	@PostConstruct와 @Transactional는 분리가 필요하다 (스프링 라이프사이클 때문에)

---

### **7_실무활용 스프링 데이터 JPA와 Querydsl**

#### 7.1 스프링 데이터 JPA 리포지토리로 변경

-	JpaRepository 상속받으면 됨

#### 7.2 사용자 정의 리포지토리

-	사용자 정의 리포지토리 사용법
	-	사용자 정의 인터페이스 작성
	-	사용자 정의 인터페이스 구현
	-	스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

#### 7.3 스프링 데이터 페이징 활용1 - Querydsl 페이징 연동

-	스프링 데이터의 Page, Pageable 을 활용

-	querydsl에서 카운트 한번에 구하기 vs 따로 분리하기

	-	fetchResult() vs fetchCount()

#### 7.4 스프링 데이터 페이징 활용2 - CountQuery 최적화

-	스프링 데이터 라이브러리가 제공함
-	count 쿼리가 생략 가능한 경우 생략함

	-	페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
	-	마지막 페이지일 때(offset + 컨텐츠 사이즈를 더해서 전체사이즈 구함)

-	return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);

#### 7.5 스프링 데이터 페이징 활용3 - 컨트롤러 개발

-	-	스프링 데이터 정렬은 sort를 querdsl의 정렬(OrderSpecifier) 로 편리하게 변경하는 기능을 제공함

	-	정렬은 조건이 조금만 복잡해져도 Pageable의 Sort 기능을 사용하기 어려움
		-	루트 엔티티 범위를 넘어가는 동적 정렬 기능이 필요하면 스프링 데이터 페이징이 제공하는 Sort를 사용하기 보다는 파라미터를 받아서 직접 처리하는 것을 권장함

---

### **8_스프링 데이터 JPA가 제공하는 Querydsl 기능**

-	8장의 기능들이 제약이 커서 복잡한 실무환경에서 사용하기에 제약이 많다

#### 8.1 인터페이스 지원 - QuerydslPredicateExecutor

-	한계

	-	조인X (묵시적 조인은 가능하지만 left join이 불가능하다)
	-	클라이언트가 Querydsl에 의존해야 한다. 서비스 클래스가 Querydsl이라는 구현 기술에 의존해야 한다.
	-	복잡한 실무환경에서 사용하기에는 한계가 명확하다.

#### 8.2 Querydsl Web 지원

-	외부에서 들어오는 파라미터를 @QuerydslPredicate 를 통해 받을 수 있음

-	한계

	-	단순한 조건만 가능
	-	조건을 커스텀하는 기능이 복잡하고 명시적이지 않음
	-	컨트롤러가 Querydsl 에 의존
	-	복잡한 실무환경에서 사용하기에는 한계가 명확

#### 8.3 리포지토리 지원 - QuerydslRepositorySupport

-	장점

	-	getQuerydsl().applyPagination() 스프링 데이터가 제공하는 페이징을 Querydsl로 편리하게 변환 가능(sort 는 오류 발생...)
	-	from 으로 시작가능
	-	EntityManager 제공

-	단점

	-	querydsl 3.x 을 대상으로 만들어짐
	-	querydsl 4.x 에 나온 JPAQueryFactory로 시작할 수 없음

		-	select로 시작할 수 없음

	-	QueryFactory를 제공하지 않음

	-	스프링 데이터 Sort 기능이 정상 동작하지 않음

#### 8.4 Querydsl 지원 클래스 직접 만들기

-	querydslRepositorySupport 가 지닌 한계를 극복하기 위한 직접 Querydsl 지원 클래스를 만든다

-	장점

	-	스프링 데이터가 제공하는 페이징 편리하게 변환
	-	페이징과 카운트 쿼리 분리 가능
	-	스프링 데이터 Sort 지원
	-	select(), selectFrom() 으로 시작 가능
	-	EntityManager, QueryFactory 제공
