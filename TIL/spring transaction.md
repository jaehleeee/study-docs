# 스프링 트랜잭션

## 하나의 비지니스 로직을 하나의 트랜잭션에 담으려면?
### 아래 순서대로 비지니스 로직을 넣는다.
#### JDBC 트랜잭션 사용 수도 코드
```
 connection = getConnection
 connection.setAutoCommit(false)
 {비지니스 로직} with connection // 비지니스 로직에서 트랜잭션 사용시 이 커넥션을 보게 해야함.
 connection.commit()
 connection.setAutoCommit(true)
 connection.close()
```

#### JPA 트랜잭션 사용 수도 코드
```
//엔티티 매니저 팩토리 생성
EntityManagerFactory emf =Persistence.createEntityManagerFactory("jpa-test");
EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성
EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득
    
try {
    tx.begin(); //트랜잭션 시작 
    logic(em); //비즈니스 로직
    tx.commit();//트랜잭션 커밋
} catch (Exception e) { 
    	tx.rollback(); //트랜잭션 롤백
} finally {
	em.close(); //엔티티 매니저 종료
}

	emf.close(); //엔티티 매니저 팩토리 종료
}
```


### 이렇게 할 경우 문제점
 * 코드가 깔끔하지 않다.
 * 데이터 액세스 기술에 의존적인 코드가 된다 (JDBC냐? JPA냐?)
 * 비즈니스 로직과는 다른 관심사의 일에 에너지를 써야 한다.

### 트랜잭션 추상화
 * 문제: 구현 기술에  따라 트랜잭션 사용 기술이 다르다
   * JDBC : con.setAutoCommit(false)
   * JPA : transaction.begin()
 * 이 문제를 해결하려면 트랜잭션 기능을 추상화(인터페이스)하면 된다.
 * 스프링은 트랜잭션 기능을 추상화화여 제공하기 위해 PlatformTransactionManager 인터페이스를 제공한다.
 * 그리고 이 인터페이스를 베이스로하여 필요한 데이터 액세스 기술에 맞게 구체화한 클래스를 제공한다.

```
public interface PlatformTransactionManager extends TransactionManager {
      TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
      void commit(TransactionStatus status) throws TransactionException;
      void rollback(TransactionStatus status) throws TransactionException;
}
```

![image](https://github.com/user-attachments/assets/24fa1c53-6c3f-4d25-a8b2-faef947f87f7)


## 스프링 트랜잭션 기술
### 트랜잭션 동기화
 * 트랜잭션을 비지니스 로직에서 잘 동작하게 하려면 connection 등의 리소스를 잘 동기화 시켜야 한다.
 * 이를 파라미터를 계속 들고다니면서 옮겨주기엔 너무 어글리하다.
 * 스프링은 이를 해결하기 위해서 트랜잭션 동기화 매니저를 제공한다.
    * 쓰레드 로컬을 사용하여 커넥션을 동기화시켜준다.
    * 쓰레드 로컬을 사용하기 때문에 멀티쓰레드 상황에서도 안전하게 동기화 가능하다.

```
TransactionSynchronizationManager.initSynchronization();
Connection connection = DateSourceUtils.getConnection(dataSource); // 현재 스레드에서 사용하게 될 커넥션을 스레드에 바인딩한다.
connection.setAutoCommit(false);
 {비지니스 로직} without connection
 connection.commit()
 connection.setAutoCommit(true)
 connection.close()

```

![image](https://github.com/user-attachments/assets/7100432b-dfb4-4128-8c35-76d0823e4768)



### 선언적 트랜잭션 @Transactional
 * @Transactional AOP를 통해 구현
 * 타겟 메서드를 가진 클래스를 상속 받아 트랜잭션 경계로 감싼 프록시로 만든다.

## 스프링 트랜잭션 속성
 * propagation
 * isolation
 * read-only
 * timeout
 * rollback-for
 * no-rollback-for





## 참고 링크
 * https://youtu.be/cc4M-GS9DoY?si=hlCM4IK6jdPGQb7N
 * https://velog.io/@daehoon12/Spring-DB-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EB%A7%A4%EB%8B%88%EC%A0%80
