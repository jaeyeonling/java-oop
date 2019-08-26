# 캡슐화 (Encapsulation)

* 데이터 + 관련 기능 묶기
* 객체가 기능을 어떻게 구현했는지 외부에 감추는 것
  * 구현에 사용된 데이터의 상세 내용을 외부에 감춤
* 정보 은닉 (Information Hiding) 의미 포함
* 외부에 영향 없이 객체 내부 구현 변경 가능

## 캡슐화를 하지 않으면?

* 요구사항의 변화가 데이터 구조/사용에 변화를 발생시킨다.

## 캡슐화를 하면?

* 기능을 제공하고 구현 상세를 감춘다.
* 기능 조건 변화
  * 내부 구현만 변경된다.
* 캡슐화는 연쇄적인 변경 전파를 최소화할 수 있다.
  * 캡슐화된 기능을 사용하는 코드 영향 최소화

## 캡슐화와 기능

* 캡슐화를 시도 -> 기능에 대한 이해(의도)를 높인다.

## 캡슐화를 위한 규칙

* Tell, Don't Ask

  * 데이터를 달라하지 말고 해달라고 하기

  ```java
  // before
  if (account.getMembership() == REGULAR) {
  	// ...
  }
  
  // after
  if (account.hasRegularPermission()) {
  	// ...
  }
  ```

* Demeter's Law

  * 메서드에서 생성한 객체의 메서드만 호출
  * 파라미터로 받은 객체의 메서드만 호출
  * 필드로 참조하는 객체의 메서드만 호출

  ```java
  // before
  account.getExpDate().isAfter(now);
  
  // after
  account.isExpired();
  ```

## 정리

* 캡슐화: 기능의 구현을 외부에 감춘다.
* 캡슐화를 통해 기능을 사용하는 코드에 영향을 주지 않고 (또는 최소화) 내부 구현을 변경할 수 있는 유연함

## 연습

### Q1. 아이디 일치 여부 확인

```java
// before
public AuthResult authenticate(final String id, final String pw) {
  final Member member = findOne(id);
  if (Objects.isNull(member)) {
    return AuthResult.NO_MATCH;
  }
  if (member.getVerificationEmailStatus() != 2) {
    return AuthResult.NO_EMAIL_VERIFIED;
  }
  if (passwordEncoder.isPasswordValid(member.getPassword(), pw, member.getId())) {
    return AuthResult.SUCCESS;
  }
  
  return AuthResult.NO_MATCH;
} 

// after
public AuthResult authenticate(final String id, final String pw) {
  final Member member = findOne(id);
  if (Objects.isNull(member)) {
    return AuthResult.NO_MATCH;
  }
  if (member.isEmailVerified()) {
    return AuthResult.NO_EMAIL_VERIFIED;
  }
  if (member.matchPassword(passwordEncoder, pw)) {
    return AuthResult.SUCCESS;    
  }
  
  return AuthResult.NO_MATCH;
} 
```



### Q2. 영화 조건에 맞춰서 렌탈 여부확인 

```java
// before
public class Rental {
  private Movie movie;
  private int daysRented;
  
  public int getFrequentRenterPoints() {
    if (movie.getPriceCode() == Move.NEW_RELEASE && daysRented > 1) {
      return 2;
    }
    
    return 1;
  }
}

public class Movie {
  public static int REGULAR = 0;
  public static int NEW_RELEASE = 1;
  
  private int priceCode;
  
  public int getPriceCode() {
    return priceCode;
  }
}

// after
public class Rental {
  private Movie movie;
  private int daysRented;
  
  public int getFrequentRenterPoints() {
    return movie.getFrequendRenterPoints(dayRented);
  }
}

public class Movie {
  public static int REGULAR = 0;
  public static int NEW_RELEASE = 1;
  
  private int priceCode;
  
  public int getPriceCode() {
    return priceCode;
  }
  
  public int getFrequentRenterPoints(final int daysRented) {
    if (priceCode == NEW_RELEASE && daysRented > 1) {
      return 2;
    }
    
    return 1;
  }
}
```



### Q3. 타이머

```java
// before
Timer timer = new Timer();
timer.startTime = System.currentTimeMillis();

// ...

timer.stopTime = System.currentTimeMillis();

final long elaspedTime = timer.stopTime - timer.startTime;

public class Timer {
  public long startTime;
  public long stopTime;
}

// after
Timer timer = new Timer();
timer.start();

// ...

timer.stop();

final long elaspedTime = timer.elapsedTime();

public class Timer {
  public long startTime;
  public long stopTime;
  
  public void start() {
    timer.startTime = System.currentTimeMillis();
  }
  
  public void stop() {
    timer.stopTime = System.currentTimeMillis();
  }
  
  public long elapsedTime() {
    return stopTime - startTime;
  }
}


```



### Q4. 이메일 검증

```java
// before
public void verifyEmail(final String token) {
  final Member member = findByToken(token);
  if (Objects.isNull(member)) {
    throw new BadTokenException();
  }
  if (member.getVerificationEmailState() == 2) {
    throw new AlreadyVerifiedException();
  } else {
    member.setVerificationEmailStatus(2);
  }
  
  // ...
}

public class Member {
  private int verificationEmailStatus;
  
  public void setVerificationEmailStatus(final int verificationEmailStatus) {
    this.verificationEmailStatus = verificationEmailStatus;
  }
  
  public int getVerificationEmailState() {
    return this.verificationEmailStatus;
  }
}

// after
public void verifyEmail(final String token) {
  final Member member = findByToken(token);
  if (Objects.isNull(member)) {
    throw new BadTokenException();
  }
  
  member.verifyEmail();
  // ...
}

public class Member {
  private int verificationEmailStatus;
  
  public void verifyEmail() {
	  if (isEmailVerified()) {
     throw new AlreadyVerifiedException();
		}
    
    this.verificationEmailStatus = 2;
  }
  
  public boolean isEmailVerified() {
    return verificationEmailStatus == 2;
  }
}
```
