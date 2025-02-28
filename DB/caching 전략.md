### 1. Cache Aside (Lazy Loading) 전략

<img src="/images/0228/look-aside.png" alt="">

Cache Aside는 가장 일반적으로 사용되는 캐싱 패턴이다.

1. application이 먼저 캐시를 확인한다.
2. 캐시에 데이터가 없으면(Cache Miss) DB에서 원본 데이터를 조회한다.
3. DB에서 가져온 데이터를 캐시 스토어에 추가한다.

특징:

- application이 캐시와 데이터 스토어 모두를 직접 관리한다.
- 필요한 데이터만 캐시에 로드되어 메모리 효율적으로 사용된다.

단점:

- 첫 요청 시 Cache Miss가 발생하여 응답 속도가 느리다.
- 데이터 갱신 시 캐시 무효화 로직이 필요하다.

적용 사례:

- 읽기 작업이 많고 쓰기가 적은 경우
- 웹 application, 세션 정보 저장

### 2. Read-Through 전략

<img src="/images/0228/read-through.png" alt="">

Read-Through는 Cache Aside와 유사하나 저장하는 주체가 서버가 아닌 DB이다.

1. application이 캐시에만 데이터를 요청한다.
2. 캐시에 데이터가 없으면 캐시 계층이 자동으로 DB에서 데이터를 로드한다.
3. 캐시 계층이 로드한 데이터를 캐시 스토어에 저장하고 애플리케이션에 반환한다.

특징:

- 캐시 계층이 데이터 소스와의 상호작용을 관리한다.
- application 코드가 단순해진다(캐시 로직이 분리됨).

단점:

- Cache Aside와 마찬가지로 첫 요청이 느리다.
- 캐시 라이브러리/서비스가 이 패턴을 지원해야 한다.

적용 사례:

- 분산 캐시 시스템
- 마이크로서비스 아키텍처

### 3. Write-Through 전략

<img src="/images/0228/write-through.png" alt="">
Write-Through는 데이터 일관성을 우선시하는 패턴이다.

1. 데이터 쓰기 시 먼저 캐시에 기록한다.
2. 캐시는 즉시 데이터를 DB에 저장한다.
3. 읽기 요청 시에는 항상 캐시에서 읽는다.

특징:

- 캐시와 DB의 데이터가 항상 동기화된다.
- 종종 Read-Through와 함께 사용된다.

단점:

- 모든 쓰기 작업에 지연 시간이 추가된다.
- 사용되지 않을 수 있는 데이터까지 캐시에 저장된다.

적용 사례:

- 데이터 일관성이 중요한 금융 시스템
- 읽기/쓰기 비율이 비슷한 경우

### 4. Write-Behind (Write-Back) 전략

<img src="/images/0228/write-back.png" alt="">
Write-Behind는 성능 최적화에 중점을 둔 패턴이다.

1. 데이터 쓰기 시 먼저 캐시에만 저장한다
2. 일정 시간 후 또는 조건 충족 시 캐시의 변경사항을 배치로 DB에 반영한다.

특징:

- 쓰기 작업을 최적화하여 시스템 처리량이 향상된다.
- 여러 쓰기 작업을 병합하여 DB 부하를 줄인다.

단점:

- 캐시 장애 시 아직 DB에 반영되지 않은 데이터가 유실될 수 있다.
- 데이터 일관성이 일시적으로 보장되지 않는다.

적용 사례:

- 로그 수집 시스템
- 높은 쓰기 처리량이 필요한 시스템
- 분석 데이터 처리

### 전략 선택 시 고려사항

- 데이터 접근 패턴: 읽기 중심인지 쓰기 중심인지에 따라 적합한 전략이 달라진다.
- 일관성 요구사항: 강한 일관성이 필요하면 Write-Through, 최종 일관성이 허용되면 Write-Behind가 적합하다.
- 응답 시간 요구사항: 일관된 응답 시간이 중요하면 Read-Through 또는 Cache Aside + 사전 워밍이 좋다.
- 시스템 복잡도: 단순한 구현이 필요하면 Cache Aside, 고급 기능이 필요하면 다른 전략을 고려한다.

### 전략 비교

| 전략            | 읽기 성능 | 쓰기 성능 | 데이터 일관성 | 구현 복잡도 | 데이터 유실 위험 |
|---------------|-------|-------|---------|--------|-----------|
| Cache Aside   | 중간    | 빠름    | 낮음      | 낮음     | 낮음        |
| Read-Through  | 중간    | -     | 낮음      | 중간     | 낮음        |
| Write-Through | 빠름    | 느림    | 높음      | 중간     | 매우 낮음     |
| Write-Behind  | 빠름    | 매우 빠름 | 낮음      | 높음     | 높음        |

실제 시스템에서는 하나의 전략만 사용하기보다 여러 전략을 조합하여 사용하는 경우가 많다.

예를 들어, Read-Through와 Write-Through를 함께 사용하거나, Read-Through와 Write-Behind를 조합하여 사용할 수 있다.

---

#### 출처 및 참고


[쏘카의 대규모 인증토큰 트래픽 대응 : 개발기](https://tech.socarcorp.kr/dev/2023/06/27/handling-authentication-token-traffic-01.html)

[NHN CLOUD/개발자를 위한 레디스 튜토리얼](https://meetup.nhncloud.com/posts/225)

[무형상품 서비스에 캐시 적용하기](https://oliveyoung.tech/2022-12-07/oliveyoung-elasticache-springboot/)
