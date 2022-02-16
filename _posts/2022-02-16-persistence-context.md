---
layout: post
title: "Persistence Context"
date: 2022-02-16 19:47:00 +0900
categories: jekyll update
---
# Persistence Context

JPA 에서 가장 중요한 2가지

- 객체와 관계형 데이터베이스 **매핑**하기 (Object Relational Mapping)
- **Persistence Context** : Entity 를 영구 저장하는 환경
    - 눈에 보이지 않는 논리적인 개념
    - Entity Manager 를 통해서 Persistence Context 에 접근

Entity 의 생명 주기

1. 비영속 (new/transient) : Persistence Context 와 전혀 관계가 없는 **새로운** 상태
2. 영속 (managed) : Persistence Context 에 의해 **관리**되는 상태
3. 준영속 (detached) : Persistence Context 에 저장되었다가 **분리**된 상태
4. 삭제 (removed) : **삭제**된 상태

```java
// 비영속
Member member = new Member();
member.setId(1L);
member.setName("HelloJPA");

// 영속
em.persist(member);

// 준영속
em.detach(member);

// 삭제
em.remove(member);
```

### Persistence Context 의 이점

- [1차 캐시](https://www.notion.so/Persistence-Context-8dcb4e1e4d254d82b06436e8a48fde6b)
- [동일성 (identity) 보장](https://www.notion.so/Persistence-Context-8dcb4e1e4d254d82b06436e8a48fde6b)
- [Transaction 을 지원하는 쓰기 지연 (transactional write-behind)](https://www.notion.so/Persistence-Context-8dcb4e1e4d254d82b06436e8a48fde6b)
- [변경 감지 (Dirty Checking)](https://www.notion.so/Persistence-Context-8dcb4e1e4d254d82b06436e8a48fde6b)
- 지연 로딩 (Lazy Loading)

### 1차 캐시

```java
Member member = new Member();
member.setId("member1");
member.setName("회원1");

// 1차 캐시에 저장됨
em.persist(member);

// 1차 캐시에서 조회
// 1차 캐시에 존재하므로 반환
Member findMember = em.find(Member.class, "member1");
```

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled.png)

```java
// 1차 캐시에서 조회
// 1차 캐시에 없으면 DB 에서 조회하여 1차 캐시에 저장
// 1차 캐시에 존재하므로 반환
Member findMember = em.find(Member.class, "member2");

// 1차 캐시에서 조회
// 1차 캐시에 존재하므로 반환
Member findMember2 = em.find(Member.class, "member2");
```

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled%201.png)

1차 캐시가 저장되는 **EntityManager 는 Application 전체에 공유되는 것이 아니라 한 Transaction 이 끝날 때, 같이 종료되기 때문에** 비즈니스 로직이 매우 복잡하거나 거대하지 않는 이상, 1차 캐시는 **큰 이점이 아니라고 봐도 무방**

### 동일성 보장

- 1차 캐시가 마치 Collection 과 같은 역할을 해주어 객체가 같은 주소 값을 가질 수 있도록 함

```java
// DB 에서 조회
Member a = em.find(Member.class, "member1");
// a 가 DB 에서 조회한 후 1차 캐시에 저장해주었기 때문에 1차 캐시에서 조회
Member b = em.find(Member.class, "member1");

System.out.println(a == b); // true
```

### Transaction 을 지원하는 쓰기 지연

```java
em.persist(memberA);
em.persist(memberB);
// 여기까지도 INSERT SQL 을 데이터베이스에 보내지 않음

// Commit 하는 순간 데이터베이스에 INSERT SQL 을 보냄
tx.commit();
```

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled%202.png)

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled%203.png)

데이터 변경이 이루어지면 1차 캐시에 저장하면서 INSERT SQL 을 생성하여 쓰기 지연 SQL 저장소에 저장

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled%204.png)

Transaction 을 Commit 하는 순간에 Flush 가 되면서 Commit

### 변경 감지 (Dirty Checking)

```java
// 1차 캐시에 없으므로 DB 에서 조회
// DB 에서 조회된 값을 Snapshot (최초 1차 캐시에 저장될 때의 상태) 에 저장
Member findMember = em.find(Member.class, 1L);
// 데이터 변경
// Entity 와 Snapshot 을 비교
// 변경 사항이 존재하면 UPDATE SQL 을 생성하여 SQL 저장소에 저장
findMember.setName("HelloJPA2");

// Commit 하는 순간 SQL 저장소에 있는 내용을 데이터베이스로 보냄
tx.commit();
```

![Untitled](Persistence%20Context%2041a65bd0692d490499c085e9aec86b8f/Untitled%205.png)