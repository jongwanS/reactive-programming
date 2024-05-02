-  TCP 컨넥션을 맺은 상태로 통신하는지 확인
  - TIME_WAIT : tcp 컨넥션 종료,  소켓 회수 이전
  - TIME_WAIT에서 일정 시간이 지나면 CLOSED  
  ![image](https://github.com/jongwanS/reactive-programming/assets/30585897/eb12aae0-a908-416f-9ca9-e211b4082168)

- tcp connection 유지시 timewiat
  - ![image](https://github.com/jongwanS/reactive-programming/assets/30585897/067a7796-6cb6-4f7e-a172-eb9e3cf802a6)  
- 요청마다 tcp connection 을 맺을시 timewiat
  - ![image](https://github.com/jongwanS/reactive-programming/assets/30585897/aaabbf19-3a61-4452-aadd-28cbdbf74d62)  

