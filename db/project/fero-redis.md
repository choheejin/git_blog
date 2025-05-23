# 레디스 활용: 실시간 랭크 기반 랜덤 매칭 시스템

## 활용 이유

1. 초고속 데이터 처리가 가능하다.
2. 동시성 문제 해결이 가능하다.
3. Redis 자료 구조를 활용할 수 있다.
   1. Sorted Set 자료구조 활용 가능
4. 오래된 데이터를 자동으로 삭제가 가능하다.

## 고려 포인트 1. EventPublish, Redis의 Pub/Sub 무엇이 더 효율적인가

* EventPublish는 메모리에서 바로 처리하는 반면, Redis의 Pub/Sub는 네트워크의 통신을 필요로 하기 때문에 약간 느린 경향이 있다.
* EventPublish는 기본적으로 동기적으로 처리한다.
* 싱글 인스턴스 내에서만 이벤트를 처리한다면, Spring EventPublish가 더 간단하다.
* 하지만, 멀티서버 환경에서 이벤트를 공유해야한다면, Redis Pub/Sub가 더 적합하다.
  * Spring EventPublish → 내부 매칭 로직 진행
  * Redis Pub/Sub → 다른 서버에도 이벤트 전파

## 고려 포인트 2. @Scheduled가 굳이 필요할까?

* 현재 로직은 5분마다 deleteUsers() 메서드를 실행하여 만료된 사용자를 확인하고 있다.
  * 주기적인 폴링은 리소스 낭비이며 불필요한 부하를 준다.
  * 사용자가 적을 때도 동일한 주기로 실행되어 불필요한 연산이 발생한다.
* Redis Keyspace Notifications를 활용하여 로직 개선이 가능하다.
  * Redis의 만료된 키에 대한 알림을 수신하여 deleteUsers() 로직 실행 가능
  * 새로운 사용자 입장 및 키의 만료가 수신될 때만 로직 실행이 가능해진다.

```java
@Component
public class RedisExpirationListener implements ApplicationListener<SessionExpiredEvent> {

    private final MatchingService matchingService;
    private static final Logger log = LoggerFactory.getLogger(RedisExpirationListener.class);

    public RedisExpirationListener(MatchingService matchingService) {
        this.matchingService = matchingService;
        log.info("🔄 RedisExpirationListener initialized");  // 리스너 초기화 로그
    }

    @Override
    public void onApplicationEvent(SessionExpiredEvent event) {
        String expiredKey = event.getSessionId();
        log.info("⏳ Redis Key 만료 감지: {}", expiredKey);

        String[] parts = expiredKey.split(":");
        log.info("💡 Parsed key parts: {}", Arrays.toString(parts));  // 파싱된 키 부분들 로깅

        if (parts.length == 4) {
            Long exerciseType = Long.parseLong(parts[2]);
            String userId = parts[3];
            log.info("🎯 Attempting to remove user from waiting room - exerciseType: {}, userId: {}",
                    exerciseType, userId);
            matchingService.leaveWaitingRoom(userId, exerciseType);
            matchingService.deleteUsers();
        } else {
            log.warn("⚠️ Invalid key format: {}", expiredKey);
        }
    }
}
```

## 고려 포인트 3. 많은 자료구조가 필요한 로직일까?

*   입장 순서, 점수 기반 정렬, 사용자 정보를 각각 다른 자료구조를 통해 저장하는 중이다. 해당 저장이 과연 필요한 정보일까?

    ```java
      // 유저 정보 저장
      redisTemplate.opsForHash().put(userInfoKey, userToken, waitingUser);
      // 입장 순서 큐에 추가
      redisTemplate.opsForList().rightPush(queueKey, userToken);
      // 스코어 정렬셋에 추가
      redisTemplate.opsForZSet().add(sortedKey, userToken, (double) rankScore);
      // 입장 시간 TTL 설정 (5분 만료)
      stringRedisTemplate.opsForValue().set(expireKey, "EXPIRED", Duration.ofMinutes(5));
    ```
*   입장 순서에 대한 정보는 따로 저장하지 않고, 유저 정보에 있는 대기 시간 정보를 활용하여 오래 대기한 유저가 우선적으로 매칭가능하게 로직 수정이 가능하다.

    ```java
            // 2. 후보자들의 대기 시간 정보를 가져와서 정렬
            List<WaitingUser> eligibleUsers = new ArrayList<>();
            for (Object candidateObj : candidates) {
                String candidateToken = candidateObj.toString();
                // 자기 자신은 제외
                if (candidateToken.equals(userToken)) continue;

                log.info("매칭시도자의 정보: {}", userToken);
                log.info("후보자 정보: {}", candidateObj);

                // 후보자 정보 가져오기
                Object userInfoObj = redisTemplate.opsForHash().get(userInfoKey, candidateToken);
                log.info("가져온 후보자의 정보: {}", userInfoObj);
                log.info("가져온 후보자의 클래스: {}", userInfoObj.getClass());

                WaitingUser waitingUser = (WaitingUser) userInfoObj;

                log.info("waitingUser: {}", waitingUser);
                eligibleUsers.add(waitingUser);
            }


            // 3. 대기 시간이 오래된 순으로 정렬
            eligibleUsers.sort(Comparator.comparing(WaitingUser::getJoinTime));
            log.info("후보자 정보: {}", eligibleUsers);

            // 4. 가장 오래 기다린 유저와 매칭
            if (!eligibleUsers.isEmpty()) {
                WaitingUser matchedUser = eligibleUsers.get(0);
                handleMatchSuccess(matchedUser.getUserId(), userToken, exerciseId);
                matchFound = true;
                log.info("🎯 대기 시간이 가장 긴 사용자와 매칭 성공! User: {}, Wait Time: {}",
                        matchedUser.getUserId(),
                        Duration.between(matchedUser.getJoinTime(), LocalDateTime.now()).getSeconds());
            } else {
                log.info("적합한 매칭 상대를 찾지 못했습니다 (score: {})", rankScore);
            }
    ```

## 고려포인트 4. 레디스의 자료구조를 제대로 활용하고 있는가?

*   현재의 코드를 보면, 모든 사용자 정보를 다 불러오는 것을 알 수 있다. 이는 keys 명령어와 유사하게 싱글 스레드인 Redis의 구조와 적합하지 않다.

    ```java
    List<Object> waitingUsers = redisTemplate.opsForList().range(queueKey, 0, -1);
    if (waitingUsers == null || waitingUsers.isEmpty()) continue;

    for(Object userToken : waitingUsers) {
        String expireKeyString = getUserJoinTimeKey(id, userToken.toString());

    ```
*   이 코드를 통해 후보군을 먼저 뽑은 후, 추가적으로 넣은 입장 시간을 고려한 코드를 추가 작성하여 최적화할 수 있을 것 같다.

    ```java
    Set<Object> candidates = redisTemplate.opsForZSet().rangeByScore(
            sortedSetKey,
            rankScore - 100,
            rankScore + 100);
    ```

## 결과: 약 97.41% 성능 향상

JMeter를 통한 성능 테스트를 진행한 결과, 성능이 향상된 것을 확인할 수 있다.

### 전체 데이터 출력 및 매칭 시 전체 유저 조회 로직 개선

1.  개선 전 HTTP 응답 속도

    | Label        | # Samples | Average | Min | Max  | Std. Dev. | Error % | Throughput | Received KB/sec | Sent KB/sec | Avg. Bytes |
    | ------------ | --------- | ------- | --- | ---- | --------- | ------- | ---------- | --------------- | ----------- | ---------- |
    | HTTP Request | 100       | 424     | 20  | 1137 | 340.33    | 0.00%   | 2.01037    | 0.89            | 0.93        | 455.4      |
    | TOTAL        | 100       | 424     | 20  | 1137 | 340.33    | 0.00%   | 2.01037    | 0.89            | 0.93        | 455.4      |
2.  개선 이후 HTTP 응답 속도

    | Label        | # Samples | Average | Min | Max | Std. Dev. | Error % | Throughput | Received KB/sec | Sent KB/sec | Avg. Bytes |
    | ------------ | --------- | ------- | --- | --- | --------- | ------- | ---------- | --------------- | ----------- | ---------- |
    | HTTP Request | 100       | 11      | 7   | 18  | 2.77      | 0.00%   | 12.95505   | 5.76            | 6.01        | 455.4      |
    | TOTAL        | 100       | 11      | 7   | 18  | 2.77      | 0.00%   | 12.95505   | 5.76            | 6.01        | 455.4      |

### 스케줄러 관련 로직 개선

1. 개선 전 CPU 사용률
   *   스케줄러 실행 전 cpu

       ```json
       {
         "name": "system.cpu.usage",
         "description": "The \"recent cpu usage\" for the whole system",
         "baseUnit": null,
         "measurements": [
           {
             "statistic": "VALUE",
             "value": 0
           }
         ],
         "availableTags": []
       }
       ```
   *   스케줄러 실행 후 cpu

       ```json
       {
         "name": "system.cpu.usage",
         "description": "The \"recent cpu usage\" for the whole system",
         "baseUnit": null,
         "measurements": [
           {
             "statistic": "VALUE",
             "value": 0.116052344290215
           }
         ],
         "availableTags": []
       }
       ```
2.  개선 이후 CPU 사용률

    > Http request 100개 테스트 이후 바로 실행하여 실행 전에도 cpu 사용률이 높은 모습을 볼 수 있다.

    *   스케줄러 실행 전 cpu

        ```json
        {
          "name": "system.cpu.usage",
          "description": "The \"recent cpu usage\" for the whole system",
          "baseUnit": null,
          "measurements": [
            {
              "statistic": "VALUE",
              "value": 0.0934590412650209
            }
          ],
          "availableTags": []
        }
        ```
    *   스케줄러 실행 후 cpu

        ```json
        {
          "name": "system.cpu.usage",
          "description": "The \"recent cpu usage\" for the whole system",
          "baseUnit": null,
          "measurements": [
            {
              "statistic": "VALUE",
              "value": 0.10549394596041628
            }
          ],
          "availableTags": []
        }
        ```

CPU 사용률이 약 9.09% 개선되었습니다.

<details>

<summary>개선 비율 계산식</summary>

**1. CPU 사용률 변화**

* 개선 전 CPU 사용률: ( 0.116052344290215 )
* 개선 후 CPU 사용률: ( 0.10549394596041628 )

**2. 개선 비율 계산식**

\[\
\text{개선 비율} = \left( \frac{\text{개선 전 사용률} - \text{개선 후 사용률\}}{\text{개선 전 사용률\}} \right) \times 100\
]

**3. 개선 비율 계산**

\[\
\text{개선 비율} = \left( \frac{0.116052344290215 - 0.10549394596041628}{0.116052344290215} \right) \times 100\
]

\[\
\text{개선 비율} = \left( \frac{0.01055839832979872}{0.116052344290215} \right) \times 100 \approx 9.09%\
]

</details>
