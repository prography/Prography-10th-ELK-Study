## **로그의 목적**

로그는 **현재 상태를 파악**하고 **문제를 진단**하기 위한 핵심 도구. 서비스에서 문제가 발생했을 때, **어디서 어떤 일이 있었는지**를 빠르게 파악하기 위해 반드시 필요하다.

하지만 너무 많거나 중복된 로그는 오히려 중요한 정보를 가리게 된다. 따라서 **트러블슈팅에 실질적으로 도움이 되는 로그**만 선별적으로 남기는 것이 중요하다.

---

## **어떤 로그를 남길 것인가?**

> 서비스 특성에 따라 달라질 수 있으나, 다음은 일반적으로 유용한 로그 유형입니다:
> 
1. **요청/응답 로그**
    
    어떤 요청이 들어왔고, 어떤 응답을 했는지 기록합니다.
    
2. **오류 및 예외 로그**
    
    예상치 못한 문제가 발생했을 때 상세히 남깁니다.
    
3. **사용자 활동 로그** *(법적 이슈 대비)*
    
    예: 결제, 회원 탈퇴, 약관 동의 등 주요 사용자 행동
    
4. **시스템 상태 로그**
    
    APM(Application Performance Monitoring) 도구를 통해 별도로 수집합니다.
    
5. **쿼리 로그**
    
    보통 애플리케이션 레벨에서만 남기며, RDB 자체에서 남기지 않는 경우도 많습니다.
    
6. **배치 및 스케줄링 로그**
    
    정기 작업의 실행 및 결과 확인 용도입니다.
    
7. **디버깅 로그**
    
    주로 로컬 개발환경에서만 사용합니다.
    

---

## **로그 출력 방식**

- System.out.println() 대신 Slf4j (또는 Logback, Log4j 등 로깅 프레임워크) 사용
    - System.out.println()은 내부적으로 synchronize를 사용하여 락을 걸어 성능상 단점을 가짐.
- 예외 발생 시, 단순한 메시지보다는 **요청 정보와 함께** 구체적인 맥락을 기록해야 효과적입니다.

---

## **로그 레벨 구분**

| **레벨** | **설명** |
| --- | --- |
| TRACE | 가장 상세한 로그. 요청 바디, 메서드 실행 시간 등 디버깅용 세부 정보 기록 |
| DEBUG | 개발 중 디버깅을 위한 로그. 조건에 따라 발생하는 버그 확인용 |
| INFO | 시스템의 정상적인 동작 상태 기록. 주요 비즈니스 로직의 흐름 기록에 적합 |
| WARN | 잠재적인 문제 상황. 예: 응답 속도가 기준을 초과한 경우 |
| ERROR | 심각한 문제 발생. 개발자의 개입이 필요한 수준의 예외 발생 시 사용 |
| FATAL | 시스템 운영 불가능한 치명적인 오류 (일반적으로 많이 사용되진 않음) |

> ⚠️ 예외가 발생했다고 해서 무조건 ERROR로 남기지 말고,
> 
> 
> **예외의 원인과 처리 필요성**
> 

> 예: 사용자의 잘못된 요청 → INFO 또는 WARN
> 

---

## **예외 처리 전략**

### **✔ Checked vs Unchecked Exception**

| **구분** | **Checked Exception** | **Unchecked Exception** |
| --- | --- | --- |
| 처리 강제 여부 | O (try-catch 또는 throws) | X |
| 대표 예시 | IOException, SQLException | NullPointerException, IllegalArgumentException |
| 사용 권장 상황 | 복구 가능한 예외, 외부 리소스 처리 | 잘못된 요청, 로직 오류 등 주로 사용자 입력에 의해 발생 |

> 대부분의 웹 서비스에서는
> 
> 
> **Unchecked Exception을 선호**
> 

> 잘못된 사용자 요청에 대해 시스템이 복구할 수 없으므로, 예외를
> 
> 
> **적절히 알려주는 것이 핵심**
> 

---

## **로그 고도화 팁**

- **컨트롤러에 직접 로그를 남기는 것은 반복 작업**이 될 수 있으므로, **필터(Filter)** 또는 **인터셉터(Interceptor)** 레벨에서 공통적으로 요청 로그를 수집하는 것이 효율적입니다.
- 로그 포맷에는 **요청 URL, 메서드, IP, 사용자 식별자, 요청 본문 요약** 등이 포함되면 좋습니다.
- 에러 로그에는 **Exception StackTrace와 함께** 요청의 컨텍스트 정보(요청 파라미터, 헤더 등)를 함께 남깁니다.

---

## **적용 예시 (Slf4j)**

```
@Slf4j
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public ResponseEntity<UserDto> getUser(@PathVariable Long id) {
        log.info("사용자 조회 요청 - ID: {}", id);
        try {
            return ResponseEntity.ok(userService.findById(id));
        } catch (UserNotFoundException e) {
            log.warn("사용자 조회 실패 - ID: {}", id, e);
            throw e;
        } catch (Exception e) {
            log.error("서버 오류 발생", e);
            throw new InternalServerErrorException();
        }
    }
}
```
