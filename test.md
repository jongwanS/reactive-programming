# Webflux 도입 필요성
#### 리액티브 프로그래밍
  - 비동기 & 이벤트 기반 패러다임

#### 리액티브 프로그래밍의 필요성
- 비동기 & 논블로킹 지원
- 쓰레드 풀의 딜레마
  - 쓰레드를 무조건 늘린다고 문제 해결불가
    - 컨텍스트 스위칭비용 발생
- 시스템 자원을 효율적 사용
  - 논블로킹 → 시스템 자원 반환

#### Reactive Streams
- 비동기 이벤트 기반의 프로그래밍을 위한 표준을 정의한 인터페이스
- 인터페이스 구현체 ( Reactor, Rxjava, Akka)
  - 발행-구독(Publish-Subscribe) 매커니즘  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/75dc06be-539c-4ea3-9461-aea473c71483)  

#### 전통적인 thread 모델 vs 비동기 논블로킹 모델
- thread 기반 모델 (Request per thread)  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/8011f1b9-d390-4223-8661-9eb1d46f2a1a)  
출처 : 네이버 d2 유투브

- 비동기 논블로킹 모델  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/9286059a-1557-4879-9dc4-ce2bbd35fcaa)
출처 : 네이버 d2 유투브  

#### Webflux
- Reactor 기반으로 구현된 웹 프레임워크
- 동작 흐름도
  - 메인 쓰레드풀과 별도로 **webflux 내부 쓰레드풀에서 발행&구독 처리**  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/2bb28ef4-8b4c-418f-8665-c23c95180451)
출처 : 네이버 d2 유투브  

---
#### 정말 비동기로 동작할까? (코드)
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/daae7ca3-252d-436a-bb3a-b1132768fe1d)  
````java
public class ReactorTest {

    private static final Logger log = LoggerFactory.getLogger(ReactorTest.class);
    @Test
    @DisplayName("reactor-subscribe-mono")
    public void reactorSubscribe() throws InterruptedException {
        CountDownLatch cdl = new CountDownLatch(1);
        log.info("start");
        Mono<String> mono = Mono.just("hello mono").log()
                .delayElement(Duration.ofSeconds(1))
                .doOnNext(s -> log.info("구독"))
                .doOnNext(s -> cdl.countDown());
        log.info("end");
        mono.subscribe(s -> log.info("test :::::::"+s));
        log.info("======================");

        Thread.sleep(1000);
        cdl.await();
    }

    @Test
    @DisplayName("reactor-subscribe-flux")
    public void reactorSubscribeFlux() throws InterruptedException {
        CountDownLatch cdl = new CountDownLatch(1);
        log.info("start");
        Flux<String> flux = Flux.just("hello mono","hello mono2").log()
                .delayElements(Duration.ofSeconds(1))
                .doOnNext(s -> log.info("구독"))
                .doOnNext(s -> cdl.countDown());
        log.info("end");
        flux.subscribe(s -> log.info("test :::::::" + s));
        log.info("======================");

        Thread.sleep(1000);
        cdl.await();
    }

    @Test
    @DisplayName("reactor-subscribe-block")
    public void reactorSubscribeBlock() throws InterruptedException {
        CountDownLatch cdl = new CountDownLatch(1);
        log.info("start");
        Mono<String> mono = Mono.just("hello mono").log()
                .delayElement(Duration.ofSeconds(1))
                .doOnNext(s -> log.info("구독"))
                .doOnNext(s -> cdl.countDown());
        log.info("end");
        String str = mono.block();
        log.info("test :::::::" + str);
        log.info("======================");

        Thread.sleep(1000);
        cdl.await();
    }

    @Test
    @DisplayName("reactor-subscribe-block(toStream)")
    public void reactorSubscribeBlockToStream() throws InterruptedException {
        CountDownLatch cdl = new CountDownLatch(1);
        log.info("start");
        Flux<String> flux = Flux.just("hello mono").log()
                .delayElements(Duration.ofSeconds(1))
                .doOnNext(s -> log.info("구독"))
                .doOnNext(s -> cdl.countDown());
        log.info("end");
        String str = flux.toStream().findFirst().get();
        log.info("test :::::::" + str);
        log.info("======================");

        Thread.sleep(1000);
        cdl.await();
    }
}
````
#### Webflux, 쓰레드 자원을 정말 효율적으로 쓸까?  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/f34eeae2-7809-442c-a04b-b59d2660b395)  

- 그라파냐 메트릭 정보 확인
  - JVM에서 실행 중인 실시간 스레드의 수  
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/f3a120cf-9aae-4196-84ab-14af33dcbe2a)  

- webflux-nonblocking 확대
![image](https://github.com/jongwanS/reactive-programming/assets/30585897/df253c13-c570-40d6-984e-c35f956cfd5c)

> **Webflux 의 도입 목적**  
> - Thread, CPU, Memory 의 자원을 낭비하지 않고 더 많은 요청을 처리할 수 있는 고성능 웹 어플리케이션을 개발하는 것이 목적

#### 스프링에서 어떤식으로 mono/flux 를 구독할까?
- controller 에서 mono or flux 리턴 → 프레임워크에서 구독(subscribes) 처리
````java
@GetMapping("/webFluxNonBlcking")
public Mono<Object> webFluxNonBlcking() {
    //.....
    Mono<Object> mono = httpCall();
    return mono;
}
```` 


