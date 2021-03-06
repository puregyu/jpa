### JPA
 - Java Persistence API
 - 자바 진영의 ORM 기술 표준
 - EJB(엔티티 빈)는 과거 자바 표준 ORM이었지만 성능이 워낙 떨어져 Hibernate(오픈소스)가 급부상하면서 자바 진영의 표준을 내어주게 된다.
 
### ORM
 - Object-Relational Mapping(객체-관계형 DB 매핑)
 - 객체와 관계형 데이터베이스의 패러다임 불일치를 ORM 프레임워크가 중간에서 매핑
 
### JPA 사용하는 이유
 - 생산성 
 - 객체와 관계형 데이터베이스의 패러다임 불일치를 해결
   - 상속관계  
        - 개발 영역   
        ```jpa.persist(delivery);```  
        - jpa의 영역  
        ```INSERT INTO ORDER ...```  
        ```INSERT INTO DELIVERY ...```
    - 연관관계, 객체 그래프 탐색
        ```
        Member member = jpa.find(Member.class, member2Id);
        Team team = member.getTeam();
        ```
 - JPA 성능 최적화 기능
    - 1차 캐시와 동일성 보장
        - 같은 트랙잭션 안에서는 같은 엔티티 반환
    - 트랜잭션을 지원하는 쓰기 지연 (INSERT)
    - 지연 로딩과 즉시 로딩
----
### JPA에 대한 이해
#### 영속성 컨텍스트
 - `Entity` 를 영구 저장하는 환경을 의미
 - `EntityManagerFactory`는 client의 요청 당 하나의 `EntityManager`를 생성하고 `EntityManager`는 DB connection을 맺어 DB관련 처리를 수행한다.
 - `EntityManager` 를 통해 논리적인 개념인 영속성 컨텍스트에 접근한다.

#### 엔티티의 생명주기
 - 비영속(new/transient)
    - 영속성 컨텍스트에 접근하지 못한 상태
    ```
   // 객체가 생성까지만 된 상태
   Member m = new Member();
   m.setId(1L);
   m.setName("devyu");
   ```
 - 영속(managed)
    ```
    // 객체가 생성까지만 된 상태
    Member m = new Member();
    m.setId(1L);
    m.setName("devyu");
   
    EntityManager em = emf.createEntityManager();
    em.getTransaction.begin();
    em.persist(m); // 영속상태
    em.commit(); // 트랜잭션 커밋되는 순간에 SQL이 실행된다.
    ```
 - 준영속(detached)
     ```
     // 객체가 생성까지만 된 상태
     Member m = new Member();
     m.setId(1L);
     m.setName("devyu");
    
     EntityManager em = emf.createEntityManager();
     em.getTransaction.begin();
     em.persist(m); // 영속상태
     em.detach(m); // 준영속상태(회원 엔티티를 영속성 컨텍스트에서 분리)
     ```
 - 삭제(removed)
 
 #### 영속성 컨텍스트가 존재하는 이유
 - 1차 캐시
     ```
     // 비영속상태, 객체가 생성까지만 된 상태
     Member m = new Member();
     m.setId(1L);
     m.setName("devyu");
    
     EntityManager em = emf.createEntityManager();
     em.getTransaction.begin();
     em.persist(m); // 영속상태 -> 1차 캐시에 맵(Map)타입으로 저장됨(key : pk, value : entity)
     
     // db가 아니라 먼저 1차 캐시에서 조회해서 반환
     Member findMember1 = em.find(Member.class, "1L");
   
     // 1차 캐시에 없으므로 db를 조회하고 1차 캐시에 저장해놓고 반환
     Member findMember2 = em.find(Member.class, "2L");
     
     > 보통 트랜잭션 단위로 영속성컨텍스트(1차 캐시)가 생성되므로 사실 성능의 이점을 얻지는 못함.
     ```
    - 영속성 컨텍스트는 내부에 1차캐시가 존재한다.
 - 영속된 엔티티의 동일성 보장
 - 트랜잭션을 지원하는 쓰기 지연
    - 영속성 컨텍스트 내부에 쓰기 지연 SQL저장소가 존재함
    - `commit()` 을 하는 순간 쓰기 지연 SQL저장소에 쌓여있던 SQL이 `flush` 된다.
    ```
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();
   
    //엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
    transaction.begin();
   
    em.persist(memberA);
    em.persist(memberB);
    //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
   
    //커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
    transaction.commit();
    ```
 - 변경감지(dirty cheching)
    - 변경감지는 `flush()` 가 호출되면 1차 캐시에 저장되어 있는 Entity와 Snapshot을 비교하여 변경된 부분이 있는지 검사하고 변경된 것이 있으면 업데이트
    ```
   EntityManager em = emf.createEntityManager();
   EntityTransaction transaction = em.getTransaction();
   
   transaction.begin();
   
   // 영속 엔티티 조회
   Member memberA = em.find(Member.class, "memberA");
   // 영속 엔티티 데이터 수정
   memberA.setUsername("hi");
   memberA.setAge(10);
   //em.update(member) 이런 코드가 필요없다. 자바 컬렉션 다루듯이...
   
   transaction.commit();
    ```
 - 지연로딩
 
#### flush
 - 영속성 컨텍스트의 변경내용을 DB에 전달(commit 전까지 실제로 변경되진 않음)
 - 보통 DB 트랜잭션이 commit 될 때, flush가 호출됨
 - 변경감지 / 수정된 엔티티 쓰기지연 SQL 저장소 등록 -> 쓰기 지연 SQL 저장소에 등록된 쿼리를 DB에 전달
 - 언제 flush 되니?
    - `emtityManager.flush()`, 직접호출
    - 트랜잭션 커밋, 자동호출
    - JPQL 쿼리 실행, 자동호출
 - flush()는 영속성 컨텍스트의 1차 캐시를 비우는 것이 아니라 쓰기 지연 SQL 저장소에 쌓여있는 SQL을 반영시키는 역할을 하는 것...
 
#### 기본 키(pk) 매핑 전략
 @GeneratedValue
 - strategy = GenerationType.AUTO
    - default 전략
    - db dialect에 따라 자동으로 생성
 - strategy = GenerationType.IDENTITY
    - 기본 키 생성을 db에게 위임, mysql
    - `IDENTITY` 전략의 경우, 실제 db에 데이터가 들어가기 전까지 pk값을 알 수가 없음
    - jpa에서 영속성 컨텍스트(1차 캐시)는 pk : entity 형식으로 존재해야하니깐... 기본키 매핑전략중에 예외적으로
    영속상태가 되자마자 db query가 바로 수행된다. 쿼리가 수행되어 pk값이 특정지어 지면 jpa는 내부적으로 pk값을 select 해와서
    영속성 컨텍스트(1차 캐시)에 넣어준다. 쓰기지연SQL 저장소에서 버퍼링 기능을 사용하지 못하게 된다.(사실 버퍼링이 시스템적으로 큰 효율은 아님)
    ```
    Member m = new Member();
    m.setName("devyu");
    em.persist(m); // commit하기 전에 해당라인에서 insert query가 수행되어 pk값이 db에 생성된다.
    log.info("id = {}", m.getId()); // 생성된 pk는 1차캐시에 바로 담아두기 때문에 id값 확인도 가능하다.
    
    em.commit();
    ```
 - strategy = GenerationType.SEQUENCE
    - db 시퀀스 오브젝트 사용, oracle
    - 영속상태가 될 떄, 시퀀스의 값을 가져온다.
    - Entity 레벨에서 @SequenceGenerator 필요
        - allocationSize : 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨
                           데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값
                           을 반드시 1로 설정해야 한다
 - strategy = GenerationType.TABLE
    - 키 생성 전용 테이블을 만들어 시퀀스를 흉내내는 전략
    - Entity 레벨에서 @TableGenerator 필요
        - allocationSize : 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨)
         
----

### 객체 - DB 매핑
 - 단방향 연관관계
    - 단방향 연관관계로도 매핑은 완료
    - 단지 양방향 연관관계는 반대 방향으로 조회(객체 그래프 탐색)의 기능을 추가한 것 뿐
 - 양방향 연관관계 
    - 1 section : `@OneToMany` mappedBy 설정 필수
    - N section : `@ManyToOne`, `@JoinColumn` 설정 필수
    - DB와 객체간의 패러다임 불일치가 발생하는 부분으로 DB의 경우 Forignkey 만으로 테이블을 조인하여
    양방향 참조가 가능하지만 객체의 경우, 양방향 참조를 하려면 단방향 연관관계가 2개가 있는 것이다.
    - 팀 객체에서 회원 객체를 참조할 수 있는 필드가 있어야 하고.. (다대일)
    - 회원 객체에서 팀 객체를 참조할 수 있는 필드가 있어야 하고.. (일대다)
    - 그럼 객체의 양쪽에 참조필드를 만들어두었는데... 양쪽 중에 어떤게 바뀌었을 때, 업데이트 되어야 하지?
    ```
    @Entity
    class Member {
      @Id @GeneratedValue
      private Long id;
      
      @ManyToOne
      @JoinColumn
      private Team team; // Member객체에서 Team객체를 참조할 수 있도록 만들고..
    }
   
   
    @Entity
    class Team {
      @Id @GeneratedValue
      private Long id;
   
      @OneToMany(mappedBy = "team")
      private List<Member> members = new ArrayList<>(); // Team객체에서 Member객체를 참조할 수 있도록 만들고..
    } 
    ```
    - Member의 team을 수정해? Team의 members를 수정해? -> 연관관계의 주인을 정하자!
    - *연관관계 주인*
        - 두 객체중 하나를 연관관계의 주인으로 지정
        - 주인으로 지정된 객체에서 외래키를 관리(등록, 수정)
        - 주인이 아닌 쪽은 read 권한만 가짐
        - 주인이 아닌 쪽에서 mappedBy 속성을 사용하여 주인을 팔로잉함
        - 외래키가 있는 쪽이 주인(다 대 일 중에서 다(多) 쪽) !!!!!!!!!!!!!!!!!!

---
### 다양한 연관관계 매핑
1. 다대일[N:1]
    - 가장 많이 사용됨
    - `[N]`쪽이 연관관계 주인이 되는 모델
        - `[N]` 쪽에는 `@ManyToOne`, `@JoinColumn` 설정
        - 양방향 연관관계 설정까지 원하면 `[1]` 쪽에 `@OneToMany(mappedBy = "변수명")` 설정
2. 일대다[1:N]
    - `[1]`쪽이 연관관계 주인이 되는 모델
    - 권장하는 모델은 아님
    - DB와 객체의 불일치
        - DB의 경우 [N]쪽에 무조건 FK가 들어가게 되어있으나, 객체의 입장서는 [1]쪽에서 [N]쪽을 참조하는 변수가 생성(FK같이..)
        ![일대다-DB와 객체의 패러다임 불일치](./img/1-n-mapping.png)
        - Team 객체의 `members` 가 연관관계의 주인이라 해당 list가 변경되면 실제 DB에서는 Member 테이블에 업데이트가 되는 불일치
        - 엔티티가 관리하는 FK가 다른 테이블에 있는게 가장 큰 단점..
        - 연관관계 관리를 위해 추가로 UPDATE SQL 실행
 
3. 일대일[1:1]
    - 주 테이블과 대상 테이블 중에 FK 선택가능
    - FK를 설정할 엔티티에 `@OneToOne`, `@JoinColumn(name = "대상ID")` 설정을 하고 대상 엔티티에서도 참조를 한다면 `@OneToOne(mappedBy="주엔티티 조인컬럼 변수명"")`을 걸 것
    
  

