# 이벤트 기반 분산 시스템을 향한 여정

### 모놀리식 시스템에서 도메인 기반으로한 이벤트 분산 시스템으로의 여정

### Adapter 서비스의 역할 

 - 두 시스템 간의 이질적인 용어와 개념에 대해서 연결하는 역할 

 - 도메인 주도 개발에서의 반부패 계층
   - 가장 방어적인 컨텍스트 매핑 관계이다.
   - 하류 팀이 그들의 보편언어 모델과 상류 팀의 보편언어 모델 사이에 번역 계층을 만드는 것이다.
   - 이 계층은 상류 모델로부터 하류 모델을 독립시키고 둘 사이를 번역한다.
   
 - 복잡하게 얽히고 꼬인 레거시 시스템을 깨기 위한 것 

### 메세징 기반의 비동기 처리, 시스템간 결합 제거 

- 수신된 메세지에 대해서 멱등성을 보장하도록 기능 개발 
  - 동일한 주문에 대해서 중복되서 보내지 않도록 할 것 

> 멱등성 : 수학이나 전산학에서 연산의 한 성질을 나타내는 것으로, 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미한다.

### 불어나는 기능, 커지는 서비스, 쌓이는 도메인 지식

- 운영되는 업무에 의해서 쌓이는 도메인 지식 
  - 도메인의 경계가 분리되어 보이기 시작함 -> 업무를 파악했기 때문에. 

### 응집력 있는 도메인 개념 묶기 

- Package(패키지)로 모듈 구성 
  - 패턴에 따르는 구성이 아닌 도메인의 경계에 따라 분리하는 구조로 구성함. 

- 도메인 주도 개발에서의 바운디드 컨텍스트의 개념을 적용해보고자 함. 

### 왜 모듈화 인가?

- 잘 경계지어진 모듈은 단일 JVM에서 동작하는 마이크로 서비스와 같다. 
  - 모듈간 상호 작용은 내부 프로세스로 처리 
  - 단일 트랜잭션 관리로로 인해 강력한 일관성 확보 
  - IDE를 통합 손수인 리팩토링 작업 

```java

@Component
@Transactional
class Sample {
    
    @Autowired private PurchaseService purchaseService;
    @Autowired private DeliveryService deliveryService;
    @Autowired private LogisticOperations logisticOperations;
    
    public void process(OrderCompleted orderCompleted){
        PurchaseOrder order = purchaseService.createOrder(...);
        logisticOperations.register(order);
        DeliveryInvoice invoice = deliveryService.createOrder(...);
        logisticOperations.register(order);
        ...
    }   
}

```

### 또 다른 모놀리식 시스템화 현상 

- 모듈간 직접 참조에 의한 강결합이 발생하게 됨. 
- 서로의 업무 도메인간의 서로를 너무 잘알고 있어서 변경에 취약해짐. 

### 이벤트 생산과 소비를 통한 느슨한 결합 

- Event Producer (생산) -> Event Channel -> Event Consumer (소비)
  - 생산과 소비의 역할이 분리되면서 결합도가 낮아짐 

### 스프링 애플리케이션 이벤트 메커니즘

- @EventListener 
- 도메인 이벤트 기잔으로 통합되는 모듈 
  - 모델의 행위에 대해서 깊이있는 고민의 시작 

### 객체지향 

 - 강한 응집력, 느슨한 결합력 
   - 모놀리스, 마이크로리스, 분산된 모놀리식

### 시스템 또는 서비스 통합을 위한 세가지 방법 

각 요소의 장담점이 있음 

- 레스트풀 
  - 네트워크를 이용한 장애에 대한 대응 필요 
  - 트랜잭션 분기에 대한 경계 분리 고민 
- 메시징
  - 견고하게 개발이 가능함. 비동기 메세지 전달을 이용함. 
  - 의존관계도 단순하게 만들어 줄 수 있음 
  - 시스템이 복잡해질 수록 프로세스 흐름을 파악하기 어려움 
- 원격 프로시져 호출 ( RPC )
  - GRPC

### 컨텍스트 간 협업을 위한 두가지 방식 

- 동기 : 요청과 응답 ( 동기에 가까우나 비동기를 쓸 수 없는 것은 아님 (CallBack) )
- 비동기 : 이벤트 기반

### 메시징과 레스트풀 API를 적용한 시스템 아키텍처 

- 실시간 데이터 요청 : 레스트풀 API 
- 메시징 : 알림, 작업 요청 

### 메세징 시스템은 무것을 쓸 것인가?

- 최소한의 운영 비용으로 사용 할 수 있는가?
- 메세지 전달 신뢰성을 가지고 있는가?
- 단일 메시지가 여러 소비자에게 전달 될 수 있는가?
- 수평 확장성을 가지고 있는가?
- 사용하기 쉬운가?
- 모니터링 할 수 있는가?

Amazon SQS
- 최소 1일 이상 메시지 전달 보장, 최대 13일 유지 
- 무제한 확장 가능, 빠른 읽기 처리량 

Amazon SNS
- 발행/구독 모델 지원(1:1, 1:n) 
- 무제한 확장 가능 

Amazon Kinesis
- 메세지 전달 및 순서를 보장 
- 이벤트 스트리밍 지원 (중복 이벤트 제어)
- 샤드(Shard)개념을 통해서 무제한 확장 가능 

Amazon MQ
- 메세지 전달 및 순서를 보장 
- 점대점, 발행/구독 방식 메세지 기능 지원 
- 인스턴스 기반 확장 가능 

### Amazon SNS와 Amazon SQS에 협업 
- 팬 아웃 패턴을 통해 여러 구독자에게 메세지를 발행 

### Amazone SNS/SQS가 적용된 메시징 아키텍처 

### 메시지는 도메인 이벤트를 싣고...

### 스프링 메시징 모듈, 메시징 통합 지원 

### Spring Cloud Stream, 다중 바인드 지원 

### 이벤트 기반 애플리케이션과 메시징 어뎁터

- 헥사고날 아키텍처의 어댑터의 역할 수행 
  - 메시징 -> 이벤트
  - 이벤트 -> 메시징 

### 스프링 컴포넌트 기반으로 만들어진 메세징 어댑터 

### 타입정보를 사용한 JSON 직렬화/역직렬화 

- 타입 정보를 포함해 JSON 직렬화

```java 

class Sample {

    public void static main(String[] args){
        ObjectMapper objectMapper = new ObjectMapper();
        
        objectMapper.enableDefaultTyping();
    }
}

```

### 풀어야할 과제들

- 시스템 모니터링에 어려움과 해결하기 위한 노력
  - 감시와 보상 프로세스 개선
  - 이벤트 저장소, 이벤트 히스토리 추적등 ( EIP 패턴 속에서 답 찾아보기 )
- CQRS 패턴에 필요성과 도입 검토 
  - 현재는 이벤트들이 트리가가 되어 동자하는 형태이며, 아직 시스템 복잡도가 높지 않음 
  - 향후 명령형 이벤트를 비롯해 여러 시스템에 데이터를 한곳에서 봐야하는 요구 사항 
- 데이터 일관성을 확보하기 위한 노력 
  - Event Sourcing 
- 늘어나는 서비스와 인프라를 효율적으로 운용하기 위한 방안 찾기 
  - 서버리스 컴퓨팅을 기반으로 이벤트 중심 마이크로 서비스 운영 
- 서비스별 메세지 큐(SQS) 관리 비용을 낮추기 위한 방안
  - Amazon Kinesis를 통한 메세지 스트림 도입과 적용

# 참조

> [정리한 강연 자료 링크](https://www.youtube.com/watch?v=Dn5irt7bClM)


# 링크 

> [도메인 주도 개발](https://goldfishhead.tistory.com/84)  
> [메세지 큐를 이용한 비동기 요청 처리 - 이동욱](https://dongwooklee96.github.io/post/2021/03/29/%EB%A9%94%EC%8B%9C%EC%A7%80-%ED%81%90%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B9%84%EB%8F%99%EA%B8%B0%EC%B2%98%EB%A6%AC-%EB%B0%8F-%EC%97%90%EB%9F%AC-%EC%B2%98%EB%A6%AC/)  
> [Spring Framework의 Application Event 활용기 - Yoo Young-mo](https://medium.com/@SlackBeck/spring-framework%EC%9D%98-applicationevent-%ED%99%9C%EC%9A%A9%EA%B8%B0-845fd2d29f32)  
> [GRPC.io](https://grpc.io/docs/)  
> [gRPC 깊게 파고들기](https://medium.com/naver-cloud-platform/nbp-%EA%B8%B0%EC%88%A0-%EA%B2%BD%ED%97%98-%EC%8B%9C%EB%8C%80%EC%9D%98-%ED%9D%90%EB%A6%84-grpc-%EA%B9%8A%EA%B2%8C-%ED%8C%8C%EA%B3%A0%EB%93%A4%EA%B8%B0-1-39e97cb3460) 
> [EIP 패턴 - 기업 통합 패턴](https://www.slideshare.net/barunmo/20141021-40528971)