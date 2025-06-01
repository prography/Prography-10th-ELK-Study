# 예외와 로그

## 로그의 기초 이해

### Q. Checked Exception과 Unchecked Exception은 무슨 차이인가요?

#### 자주 이야기하는 오답 ❌

> "Checked Exception은 컴파일 할 때 발생하고, Unchecked Exception은 런타임에 발생하는 예외로 알고 있습니다."

- Checked Exception → 컴파일 타임? ❌
- Unchecked Exception → 런 타임? ❌

#### 올바른 답변 ✅

> "Checked Exception은 컴파일 할 때 예외에 대한 처리를 강제하고, Unchecked Exception은 예외에 대한 처리를 강제하지 않습니다."

- **Checked Exception** → 예외 처리 강제
- **Unchecked Exception** → 예외 처리 강제 안함

## Checked Exception vs Unchecked Exception

### Checked Exception 처리 방법

#### 1. try-catch 사용
```java
public static void main(String[] args) {
    try {
        throw new CheckedException();
    } catch (CheckedException e) {
        // 예외에 대한 적절한 처리
    }
}
```

#### 2. throws 선언
```java
public static void main(String[] args) throws CheckedException {
    throw new CheckedException();
}
```

### Unchecked Exception 처리

```java
public static void main(String[] args) {
    try {
        throw new UncheckedException();
    } catch (UncheckedException e) {
        // 예외에 대한 적절한 처리
    }
}
```

Unchecked Exception은 try-catch로 처리할 수는 있지만, 컴파일러가 강제하지 않습니다.

### 예외 클래스 구조

#### Checked Exception
```java
public class CheckedException extends Exception {
}
```

#### Unchecked Exception
```java
public class UncheckedException extends RuntimeException {
}
```

### Java 예외 계층 구조

![Java 예외 계층 구조](https://file.notion.so/f/f/f0fd6e78-6fbf-45e0-ae33-bcf7d8301999/5a03081e-deea-4261-b7b6-953b694f2eab/Untitled.png?table=block&id=1b9209f5-b397-806a-856a-cc264ddeb745&spaceId=f0fd6e78-6fbf-45e0-ae33-bcf7d8301999&expirationTimestamp=1748815200000&signature=F3NJrovdJrMcMCNWrPnTpsZKiuDdZb3Z5xAUWHCY_m0&downloadName=Untitled.png)


- **Throwable**: 모든 예외와 에러의 최상위 클래스
- **Error**: 시스템 레벨의 심각한 오류 (Unchecked)
- **Exception**: 일반적인 예외의 상위 클래스 (Checked)
- **RuntimeException**: 런타임에 발생하는 예외 (Unchecked)

## 어떤 예외를 사용해야 할까?

Checked Exception과 Unchecked Exception 중 어떤 것을 사용할지는 상황에 따라 결정해야 합니다:

### Checked Exception 사용 시기

#### 1. 호출자가 예외를 처리할 수 있는 경우
```java
// 파일이 없을 때 대체 파일을 사용하거나 기본값으로 처리 가능
public String readConfig() throws FileNotFoundException {
    return Files.readString(Paths.get("config.properties"));
}

// 호출하는 쪽에서 복구 로직 구현
try {
    String config = readConfig();
} catch (FileNotFoundException e) {
    // 기본 설정값 사용으로 복구
    String config = getDefaultConfig();
}
```

#### 2. 복구 가능한 예외 상황
- **네트워크 연결 실패**: 재시도 로직 구현 가능
- **파일 읽기 실패**: 대체 파일 사용 가능
- **데이터베이스 연결 실패**: 다른 DB 서버로 전환 가능

#### 3. API 사용자가 반드시 알아야 하는 예외
- **비즈니스 로직 상 예측 가능한 실패**: 잔액 부족, 권한 없음 등
- **외부 시스템 의존성 문제**: API 호출 실패, 서비스 점검 중

### Unchecked Exception 사용 시기

#### 1. 프로그래밍 오류로 인한 예외
```java
// 잘못된 인덱스 접근 - 개발자 실수
list.get(10); // IndexOutOfBoundsException

// null 참조 - 개발자 실수  
String str = null;
str.length(); // NullPointerException

// 잘못된 형변환 - 개발자 실수
Object obj = "문자열";
Integer num = (Integer) obj; // ClassCastException
```

#### 2. 복구 불가능한 예외 상황
- **메모리 부족**: OutOfMemoryError
- **스택 오버플로우**: StackOverflowError  
- **시스템 리소스 부족**: 복구보다는 시스템 재시작이 필요

#### 3. 선택적으로 처리할 수 있는 예외
```java
// 잘못된 입력값 - 호출자가 원한다면 검증 로직 추가 가능
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("나이는 음수일 수 없습니다");
    }
}
```

## 실무에서의 선택 기준

### Checked Exception을 선택하는 경우
```java
// 외부 API 호출 - 네트워크 문제로 실패할 수 있음
public ApiResponse callExternalApi() throws ApiException {
    // 재시도나 대체 API 사용 등 복구 전략이 가능
}

// 파일 업로드 - 용량 초과, 형식 오류 등 예측 가능
public void uploadFile(File file) throws FileSizeExceededException, 
                                        UnsupportedFormatException {
    // 호출자가 사용자에게 적절한 안내 메시지 표시 가능
}
```

### Unchecked Exception을 선택하는 경우
```java
// 설정값 검증 - 개발 단계에서 발견되어야 하는 오류
public void configure(String mode) {
    if (!"DEV".equals(mode) && !"PROD".equals(mode)) {
        throw new IllegalArgumentException("지원하지 않는 모드: " + mode);
    }
}

// 비즈니스 로직 위반 - 시스템 설계 문제
public void withdraw(int amount) {
    if (amount <= 0) {
        throw new IllegalArgumentException("출금액은 양수여야 합니다");
    }
}
```

## 로깅과의 연관성

### 예외별 로깅 전략

#### ERROR 레벨 - 즉시 대응이 필요한 예외
``` java
try {
    paymentService.processPayment(order);
} catch (PaymentException e) {
    // 결제 실패는 비즈니스에 직접적 영향
    log.error("결제 처리 실패 - 주문ID: {}, 사유: {}", order.getId(), e.getMessage(), e);
    // 알람 발송
}
```

#### WARN 레벨 - 주의가 필요하지만 서비스는 계속되는 예외
``` java
try {
    notificationService.sendEmail(user);
} catch (EmailException e) {
    // 이메일 발송 실패는 주요 기능에 영향 없음
    log.warn("이메일 발송 실패 - 사용자ID: {}, 재시도 예정", user.getId());
    // 재시도 큐에 추가
}
```

#### DEBUG 레벨 - 개발/디버깅용 예외 정보

``` java
try {
    String cachedData = cacheService.get(key);
} catch (CacheException e) {
    // 캐시 실패는 성능에만 영향, DB에서 조회하면 됨
    log.debug("캐시 조회 실패 - 키: {}, DB에서 조회", key);
    return database.findByKey(key);
}
```

### 로깅 시 포함해야 할 정보

#### 필수 정보
- **컨텍스트 정보**: 사용자 ID, 요청 ID, 거래 ID 등
- **예외 발생 시점**: 타임스탬프
- **예외 원인**: 구체적인 에러 메시지

#### 선택적 정보  
- **입력 파라미터**: 민감하지 않은 데이터만
- **시스템 상태**: 메모리 사용률, 연결 풀 상태 등
- **스택 트레이스**: ERROR 레벨에서만 포함

``` java
// 좋은 로깅 예시
log.error("사용자 인증 실패 - 사용자ID: {}, IP: {}, 실패사유: {}", 
          userId, clientIp, e.getMessage(), e);

// 나쁜 로깅 예시  
log.error("에러 발생", e); // 컨텍스트 정보 부족
```

예외와 로그를 효과적으로 활용하여 시스템의 안정성과 유지보수성을 향상시킬 수 있습니다.
