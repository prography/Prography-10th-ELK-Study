# 로그 레벨

## 로그의 기초 이해

### 로그 레벨 체계

로그 레벨은 다음과 같이 구성되며, 위로 갈수록 더 높은 레벨입니다:

```
FATAL  ← 더 높은 레벨
ERROR
WARN
INFO
DEBUG
TRACE  ← 더 낮은 레벨
```

## 로그 레벨을 나누면 어떤 것을 할 수 있나요?

로그 레벨별로 다른 처리 전략을 적용할 수 있습니다:

### 레벨별 알람 정책
- **ERROR/FATAL**: 1회라도 발생시 즉시 알람!
- **WARN**: 1분간 10회 이상 발생시 알람 + 일단위 리포트
- **INFO 이하**: 3일치 로그만 보관

### 레벨별 보관 정책
- 높은 레벨 로그: 장기 보관
- 낮은 레벨 로그: 단기 보관으로 스토리지 최적화

## 로그 레벨 각각의 의미와 실무 활용

### TRACE
- **용도**: 가장 세부적인 수준의 로그
- **사용 시기**: 코드의 세부적인 실행 경로를 추적할 때 사용
- **특징**: 개발 단계에서 주로 사용, 운영 환경에서는 거의 사용하지 않음
- **실무 활용**: 
  - 메서드 진입/종료 지점 추적
  - 반복문 내부 상태 확인
  - 복잡한 알고리즘의 단계별 추적

```java
public List<User> findActiveUsers() {
    log.trace("findActiveUsers() 메서드 시작");
    
    List<User> users = userRepository.findAll();
    log.trace("전체 사용자 조회 완료, 총 {}명", users.size());
    
    List<User> activeUsers = new ArrayList<>();
    for (User user : users) {
        log.trace("사용자 {} 활성 상태 확인 중", user.getId());
        if (user.isActive()) {
            activeUsers.add(user);
            log.trace("사용자 {} 활성 상태 확인됨", user.getId());
        }
    }
    
    log.trace("findActiveUsers() 메서드 종료, 활성 사용자 {}명", activeUsers.size());
    return activeUsers;
}
```

### DEBUG
- **용도**: 디버깅 목적의 로그
- **사용 시기**: 개발 중 코드의 상태나 흐름을 이해하기 위해 사용
- **특징**: 개발자가 문제를 진단할 때 필요한 정보
- **실무 활용**:
  - 변수값 확인
  - 조건문/분기문 실행 경로 확인
  - 외부 API 호출 파라미터 및 응답 확인
  - 데이터베이스 쿼리 파라미터 확인

```java
public Order processOrder(OrderRequest request) {
    log.debug("주문 처리 시작 - 요청: {}", request);
    
    // 재고 확인
    int stock = inventoryService.getStock(request.getProductId());
    log.debug("상품 {} 재고 확인: {}개", request.getProductId(), stock);
    
    if (stock < request.getQuantity()) {
        log.debug("재고 부족 - 요청: {}개, 재고: {}개", request.getQuantity(), stock);
        throw new InsufficientStockException();
    }
    
    // 주문 생성
    Order order = Order.builder()
        .productId(request.getProductId())
        .quantity(request.getQuantity())
        .userId(request.getUserId())
        .build();
    
    log.debug("주문 객체 생성 완료: {}", order);
    return orderRepository.save(order);
}
```

### INFO
- **용도**: 시스템의 정상적인 운영 상태를 나타내는 정보성 로그
- **사용 시기**: 중요한 이벤트나 상태 변화를 기록
- **특징**: 운영 중 시스템 동작을 파악하는 데 사용
- **실무 활용**:
  - **비즈니스 로직의 주요 이벤트**: 주문 완료, 결제 성공, 회원가입 등
  - **시스템 상태 변화**: 서버 시작/종료, 배치 작업 시작/완료
  - **중요한 설정값**: 서버 포트, 데이터베이스 연결 정보
  - **외부 시스템 연동**: API 호출 성공, 메시지 큐 처리

```java
public class OrderService {
    
    @PostConstruct
    public void init() {
        log.info("OrderService 초기화 완료 - 최대 동시 처리: {}건", maxConcurrentOrders);
    }
    
    public Order createOrder(OrderRequest request) {
        log.info("주문 생성 요청 - 사용자: {}, 상품: {}, 수량: {}", 
                request.getUserId(), request.getProductId(), request.getQuantity());
        
        Order order = processOrder(request);
        
        log.info("주문 생성 완료 - 주문번호: {}, 금액: {}원", 
                order.getOrderNumber(), order.getTotalAmount());
        
        // 결제 처리
        PaymentResult result = paymentService.processPayment(order);
        if (result.isSuccess()) {
            log.info("결제 완료 - 주문번호: {}, 결제수단: {}", 
                    order.getOrderNumber(), result.getPaymentMethod());
        }
        
        return order;
    }
}

@Component
public class BatchJobScheduler {
    
    @Scheduled(cron = "0 0 2 * * *")
    public void dailyReport() {
        log.info("일일 리포트 배치 작업 시작");
        
        try {
            reportService.generateDailyReport();
            log.info("일일 리포트 배치 작업 완료");
        } catch (Exception e) {
            log.error("일일 리포트 배치 작업 실패", e);
        }
    }
}
```

### WARN
- **용도**: 잠재적으로 문제가 될 수 있는 상황을 나타냄
- **사용 시기**: 시스템 운영에는 즉각적인 영향을 주지 않지만 주의가 필요한 경우
- **특징**: 예방적 모니터링에 활용
- **실무 활용**:
  - **성능 저하 징후**: 응답시간 증가, 커넥션 풀 부족
  - **비정상적 사용 패턴**: 과도한 API 호출, 비정상적 데이터
  - **설정 문제**: 권장되지 않는 설정값 사용
  - **외부 의존성 문제**: 느린 응답, 간헐적 실패

```java
public class ApiController {
    
    private static final int SLOW_RESPONSE_THRESHOLD = 5000; // 5초
    
    @GetMapping("/api/data")
    public ResponseEntity<Data> getData() {
        long startTime = System.currentTimeMillis();
        
        Data data = dataService.getData();
        
        long responseTime = System.currentTimeMillis() - startTime;
        if (responseTime > SLOW_RESPONSE_THRESHOLD) {
            log.warn("API 응답시간 임계값 초과 - 경로: /api/data, 응답시간: {}ms", responseTime);
        }
        
        return ResponseEntity.ok(data);
    }
}

public class ConnectionPoolMonitor {
    
    @Scheduled(fixedRate = 60000) // 1분마다
    public void checkConnectionPool() {
        int activeConnections = dataSource.getNumActive();
        int maxConnections = dataSource.getMaxTotal();
        
        double usageRate = (double) activeConnections / maxConnections;
        
        if (usageRate > 0.8) {
            log.warn("DB 커넥션 풀 사용률 높음 - 사용중: {}/{} ({}%)", 
                    activeConnections, maxConnections, (int)(usageRate * 100));
        }
    }
}
```

### ERROR
- **용도**: 치명적이지 않지만, 중요한 문제가 발생했음을 나타냄
- **사용 시기**: 복구가 필요하거나 실패한 작업을 추적해야 할 때 사용
- **특징**: 즉시 대응이 필요한 문제
- **실무 활용**:
  - **비즈니스 로직 실패**: 결제 실패, 주문 처리 실패
  - **외부 시스템 연동 실패**: API 호출 실패, 데이터베이스 연결 실패
  - **데이터 무결성 문제**: 중복 데이터, 필수값 누락
  - **보안 관련 이슈**: 인증 실패, 권한 없는 접근

```java
public class PaymentService {
    
    public PaymentResult processPayment(Order order) {
        try {
            return externalPaymentApi.charge(order.getTotalAmount(), order.getPaymentInfo());
        } catch (PaymentApiException e) {
            log.error("결제 처리 실패 - 주문번호: {}, 금액: {}원, 오류: {}", 
                     order.getOrderNumber(), order.getTotalAmount(), e.getMessage(), e);
            
            // 알람 발송
            alertService.sendPaymentFailureAlert(order, e);
            
            return PaymentResult.failure(e.getMessage());
        }
    }
}

public class SecurityService {
    
    public boolean authenticateUser(String username, String password) {
        try {
            return authProvider.authenticate(username, password);
        } catch (AuthenticationException e) {
            log.error("사용자 인증 실패 - 사용자: {}, IP: {}, 사유: {}", 
                     username, getCurrentClientIp(), e.getMessage());
            
            // 보안 이벤트로 기록
            securityEventLogger.logFailedAuthentication(username, getCurrentClientIp());
            
            return false;
        }
    }
}
```

### FATAL
- **용도**: 시스템 운영을 계속할 수 없을 정도로 심각한 오류
- **사용 시기**: 시스템이 중단되거나 복구 불가능한 상태일 때
- **특징**: 가장 높은 우선순위의 알람 대상
- **실무 활용**:
  - **시스템 시작 실패**: 필수 설정 파일 누락, 데이터베이스 연결 불가
  - **메모리 부족**: OutOfMemoryError 발생
  - **필수 외부 시스템 장애**: 핵심 데이터베이스 서버 다운

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        try {
            SpringApplication.run(Application.class, args);
        } catch (Exception e) {
            log.fatal("애플리케이션 시작 실패 - 시스템 종료", e);
            
            // 긴급 알람 발송
            emergencyAlertService.sendCriticalSystemFailure(e);
            
            System.exit(1);
        }
    }
}

@Component
public class DatabaseHealthChecker {
    
    @EventListener
    public void handleDatabaseConnectionFailure(DatabaseConnectionFailureEvent event) {
        if (event.isAllConnectionsFailed()) {
            log.fatal("모든 데이터베이스 연결 실패 - 서비스 중단 불가피");
            
            // 서비스 중단 알림
            serviceStatusManager.markServiceDown();
            emergencyAlertService.sendDatabaseOutage();
        }
    }
}
```

## 실무에서의 로그 레벨 선택 가이드

### 비즈니스 로직에서
- **INFO**: 주문 생성, 결제 완료, 회원가입 등 주요 비즈니스 이벤트
- **WARN**: 재고 부족 경고, 할인 한도 초과 등 주의가 필요한 상황
- **ERROR**: 결제 실패, 주문 취소 실패 등 비즈니스 로직 오류

### 시스템/인프라에서
- **INFO**: 서버 시작/종료, 설정 로드 완료
- **WARN**: 메모리 사용률 높음, 응답시간 증가
- **ERROR**: 외부 API 호출 실패, 데이터베이스 연결 오류
- **FATAL**: 시스템 시작 실패, 모든 DB 연결 실패

### 개발/디버깅에서
- **TRACE**: 메서드 진입/종료, 루프 내부 상태
- **DEBUG**: 변수값, 조건문 분기, SQL 파라미터

## 로그 살펴보기

실제 로그의 구성 요소를 살펴보면:

```
2024-11-29T00:10:30.354+09:00  INFO 9736 --- [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializ
2024-11-29T00:10:30.354+09:00  INFO 9736 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet      : Initializ
2024-11-29T00:10:30.355+09:00  INFO 9736 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet      : Completed
2024-11-29T00:10:30.521+09:00  INFO 9736 --- [nio-8080-exec-2] k.c.s.a.SimpleShortenUrlService        : shortenUrl
2024-11-29T00:11:26.845+09:00 ERROR 9736 --- [nio-8080-exec-3] k.c.s.p.GlobalExceptionHandler         : 단축 URL을
```

### 로그 구성 요소
- **시간**: `2024-11-29T00:10:30.354+09:00`
- **레벨**: `INFO`, `ERROR`
- **PID**: `9736` (프로세스 ID)
- **스레드 이름**: `[nio-8080-exec-2]`
- **패키지 + 클래스**: `k.c.s.a.SimpleShortenUrlService`
- **로그 메시지**: 실제 로그 내용

## 로그 레벨 활용 전략

### 개발 환경
- **레벨**: DEBUG 이상
- **목적**: 상세한 디버깅 정보 확인

### 테스트 환경
- **레벨**: INFO 이상
- **목적**: 시스템 동작 추적 및 문제 진단

### 운영 환경
- **레벨**: WARN 이상
- **목적**: 성능 최적화 및 중요한 이벤트만 기록

로그 레벨을 적절히 활용하면 시스템 모니터링, 문제 진단, 성능 최적화에 효과적으로 활용할 수 있습니다.
