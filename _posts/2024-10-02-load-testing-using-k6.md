---
layout: post
title: Load Testing Using K6
categories: tool
tags: tool
---

> - [https://github.com/grafana/k6](https://github.com/grafana/k6)
> - [https://k6.io/docs/](https://k6.io/docs/)

**맥 설치 방법**

```sh
// https://grafana.com/docs/k6/latest/set-up/install-k6/
brew install k6
```

**간단 예제**

```js
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 10,
  duration: '10s',
};

export default function () {
  const host = 'https://{hostname}';
  const payload = JSON.stringify({
    id: 'helloworld',
    password: 'helloworld!!@',
  });

  const params = {
    headers: {
      'Content-Type': 'application/json',
    },
  };

  const response = http.post(host + '/users/v1/sign-in', payload, params);

  check(response, {
    'status was 200': (r) => r.status === 200,
  });
}
```

**실행 결과**

```sh
$ k6 run k6-script.js

     ✓ status was 200

     checks.........................: 100.00% ✓ 247       ✗ 0
     data_received..................: 282 kB  28 kB/s
     data_sent......................: 42 kB   4.1 kB/s
     http_req_blocked...............: avg=1.7ms    min=0s       med=1µs      max=48.2ms   p(90)=2µs      p(95)=2µs
     http_req_connecting............: avg=301.65µs min=0s       med=0s       max=7.94ms   p(90)=0s       p(95)=0s
     http_req_duration..............: avg=406.81ms min=179.17ms med=399.92ms max=696.23ms p(90)=580.04ms p(95)=598.96ms
       { expected_response:true }...: avg=406.81ms min=179.17ms med=399.92ms max=696.23ms p(90)=580.04ms p(95)=598.96ms
     http_req_failed................: 0.00%   ✓ 0         ✗ 247
     http_req_receiving.............: avg=103.79µs min=13µs     med=70µs     max=1.88ms   p(90)=170.6µs  p(95)=254.69µs
     http_req_sending...............: avg=157.88µs min=27µs     med=101µs    max=2.31ms   p(90)=298.6µs  p(95)=413.49µs
     http_req_tls_handshaking.......: avg=899.4µs  min=0s       med=0s       max=28.3ms   p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=406.55ms min=179.04ms med=399.68ms max=696.07ms p(90)=579.72ms p(95)=598.62ms
     http_reqs......................: 247     24.207321/s
     iteration_duration.............: avg=409.38ms min=197.22ms med=401.31ms max=696.6ms  p(90)=584.91ms p(95)=606.99ms
     iterations.....................: 247     24.207321/s
     vus............................: 10      min=10      max=10
     vus_max........................: 10      min=10      max=10


running (10.2s), 00/10 VUs, 247 complete and 0 interrupted iterations
default ✓ [======================================] 10 VUs  10s
```

| 지표                           | 설명                               |           예시            |
|:-----------------------------|:---------------------------------|:--------------------------:|
| **checks**                   | 테스트에서 정의한 조건을 통과한 비율             |     100.00% ✓ 247 ✗ 0      |
| **data_received**            | 서버로부터 수신된 데이터 양                  |      282 kB (28 kB/s)      |
| **data_sent**                | 클라이언트에서 서버로 전송된 데이터 양            |      42 kB (4.1 kB/s)      |
| **http_req_blocked**         | 네트워크 수준에서 요청이 차단된 시간             |   avg=1.7ms, max=48.2ms    |
| **http_req_connecting**      | 서버에 연결하는 데 걸린 시간                 |  avg=301.65µs, max=7.94ms  |
| **http_req_duration**        | 요청을 보내고 응답을 받기까지 걸린 총 시간 (응답 시간) | avg=406.81ms, max=696.23ms |
| **http_req_failed**          | 실패한 HTTP 요청의 비율                  |      0.00% (모든 요청 성공)      |
| **http_req_receiving**       | 서버로부터 응답 데이터를 수신하는 데 걸린 시간       |  avg=103.79µs, max=1.88ms  |
| **http_req_sending**         | 요청을 서버로 전송하는 데 걸린 시간             |  avg=157.88µs, max=2.31ms  |
| **http_req_tls_handshaking** | TLS 핸드셰이크에 걸린 시간                 |  avg=899.4µs, max=28.3ms   |
| **http_req_waiting**         | 서버가 응답을 준비하는 데 걸린 시간             | avg=406.55ms, max=696.07ms |
| **http_reqs**                | 총 HTTP 요청 수                      |      247 (24.21 요청/초)      |
| **iteration_duration**       | 각 테스트 반복에 걸린 시간                  | avg=409.38ms, max=696.6ms  |
| **iterations**               | 총 테스트 반복 횟수                      |            247             |
| **vus (Virtual Users)**      | 동시에 실행된 가상 사용자(Virtual Users) 수  |     10 (최소 10, 최대 10)      |

**다른 부하 테스트 도구 비교**

> [https://baeji-develop.tistory.com/118](https://baeji-develop.tistory.com/118)

