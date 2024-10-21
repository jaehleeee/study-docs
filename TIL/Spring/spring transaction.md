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

#### 스프링 AOP가 프록시 방식을 사용하는 이유는?
 * 프록시 없이, 직접 타겟 객체를 참조하려고 한다면, 타겟의 원하는 위치에서 직접 aspect 클래스를 호출해야한다.
 * 그럼 타겟 클래스 안에 부가 기능을 호출하는 로직이 포함되므로 유지보수성이 떨어진다.
 * 따라서 타겟 객체를 상속하는 프록시를 만들어서 이를 해결한다. 

#### 트랜잭션 target 호출이 들어오면 진행되는 과정
1. target에 대한 호출이 들어오면 AOP proxy가 이를 가로채서(intercept) 가져온다.
2. AOP proxy에서 Transaction Advisor가 commit 또는 rollback 등의 트랜잭션 처리를 한다.
3. 트랜잭션 처리 외에 다른 부가 기능이 있을 경우, 해당 Custom Advisor에서 그 처리를 한다.
4. 각 Advisor에서 부가 기능 처리를 마치면 Target Method를 수행한다.
5. interceptor chain을 따라 caller에게 결과를 다시 전달한다.

#### 트랜잭션 프록시 동작 수도 코드
```
public class TransactionProxy{
    private final TransactonManager manager = TransactionManager.getInstance();
		...
    public void transactionLogic() {
        try {
	    manager.begin();// 트랜잭션 전처리(트랜잭션 시작, autoCommit(false) 등)
	    target.logic(); // 다음 처리 로직(타겟 비스니스 로직, 다른 부가 기능 처리 등)
            manager.commit(); // 트랜잭션 후처리(트랜잭션 커밋 등)
        } catch ( Exception e ) {
            manager.rollback(); // 트랜잭션 오류 발생 시 롤백
        }
    }
}
```


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
