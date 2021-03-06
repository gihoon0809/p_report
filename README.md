
# 개인과제(조별과제에서 이벤트 추가) - 피자주문배달 및 리뷰 시스템 

본 프로그램은 피자주문배달 시스템이다.
(추가) 리뷰 시스템이다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW


# Table of contents

- [최종개인과제 - 피자주문배달 및 리뷰 시스템](#---)
  - [서비스 시나리오](#서비스-시나리오)
  - [체크포인트](#체크포인트)
  - [분석/설계](#분석설계)
  - [구현:](#구현-)
    - [DDD 의 적용](#ddd-의-적용)
    - [폴리글랏 퍼시스턴스](#폴리글랏-퍼시스턴스)
    - [폴리글랏 프로그래밍](#폴리글랏-프로그래밍)
    - [동기식 호출 과 Fallback 처리](#동기식-호출-과-Fallback-처리)
    - [비동기식 호출 과 Eventual Consistency](#비동기식-호출-과-Eventual-Consistency)
  - [운영](#운영)
    - [CI/CD 설정](#cicd설정)
    - [동기식 호출 / 서킷 브레이킹 / 장애격리](#동기식-호출-서킷-브레이킹-장애격리)
    - [오토스케일 아웃](#오토스케일-아웃)
    - [무정지 재배포](#무정지-재배포)

# 서비스 시나리오

기능적 요구사항
1. 고객이 피자종류와 수량을 주문한다.
2. 고객이 결제한다
3. 결제가 되면 배달 시작한다
4. 고객이 주문을 취소할 수 있다
5. 주문이 취소되면 결제가 취소된다
6. 고객이 주문상태를 중간중간 조회한다

***추가 기능적 요구사항***

1. 배달이 완료되면 리뷰를 작성할 수 있다.
2. 리뷰 작성이 완료되면 쿠폰이 발급 된다.
3. 재 주문을 진행 할 수 있다.


비기능적 요구사항
1. 트랜잭션
    1. 리뷰를 통한 주문이 완료 되어야 결제가 가능하다.
2. 장애격리
    1. 쿠폰발급기능이 수행되지 않더라도 주문은 365일 24시간 받을 수 있어야 한다  Async (event-driven), Eventual Consistency
    1. 재 주문을 통한 주문시스템이 과중되면 주문을 잠시동안 받지 않고 주문을 잠시후에 하도록 유도한다  Circuit breaker, fallback
3. 성능
    1. 고객이 주문 및 리뷰에 대한 상태를 시스템에서 확인할 수 있다 CQRS
    1. 배달이 완료되면 리뷰를 작성 할 수 있다  Event driven


# 체크포인트

- 분석 설계

  - 이벤트스토밍: 
    - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
    - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
    - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
    - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?    

  - 서브 도메인, 바운디드 컨텍스트 분리
    - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른  Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
      - 적어도 3개 이상 서비스 분리
    - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
    - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
  - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처 
    - 업무 중요성과  도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
    - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
    - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
    - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
    - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

  - 헥사고날 아키텍처
    - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?
    
- 구현
  - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
    - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
    - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를 적응시킬 수 있는가?
    - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
  - Request-Response 방식의 서비스 중심 아키텍처 구현
    - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
    - 서킷브레이커를 통하여  장애를 격리시킬 수 있는가?
  - 이벤트 드리븐 아키텍처의 구현
    - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
    - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
    - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
    - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
    - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가 가능한가?

  - 폴리글랏 플로그래밍
    - 각 마이크로 서비스들이 하나이상의 각자의 기술 Stack 으로 구성되었는가?
    - 각 마이크로 서비스들이 각자의 저장소 구조를 자율적으로 채택하고 각자의 저장소 유형 (RDB, NoSQL, File System 등)을 선택하여 구현하였는가?
  - API 게이트웨이
    - API GW를 통하여 마이크로 서비스들의 집입점을 통일할 수 있는가?
    - 게이트웨이와 인증서버(OAuth), JWT 토큰 인증을 통하여 마이크로서비스들을 보호할 수 있는가?
- 운영
  - SLA 준수
    - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
    - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
    - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
    - 모니터링, 앨럿팅: 
  - 무정지 운영 CI/CD (10)
    - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명 
    - Contract Test :  자동화된 경계 테스트를 통하여 구현 오류나 API 계약위반를 미리 차단 가능한가?


# 분석/설계
    
### 이벤트 도출
* MSAEz 로 모델링한 이벤트스토밍 결과:  http://www.msaez.io/#/storming/yAObDa1Re3WS7AwhIwSMcpwZwBH2/mine/71d0b707070bab4570137c835cc7d4bb/-MLDgWUgImpfGESNNdCA

![image](https://user-images.githubusercontent.com/67869000/98326272-2ec54300-2034-11eb-92bf-2f7ff81c7cd7.png)

 도메인 서열 분리 
   
    - Core Domain:  order,  delivery, review : 핵심 서비스이며, 연간 Up-time SLA 수준을 99.999% 목표, 배포주기는 request의 경우 1주일 1회 미만, delivery의 경우 1개월 1회 미만
    
    - Supporting Domain:   statusview, coupon : 경쟁력을 내기위한 서비스이며, SLA 수준은 연간 60% 이상 uptime 목표, 
                                                배포주기는 각 팀의 자율이나 표준 스프린트 주기가 1주일 이므로 1주일 1회 이상을 기준으로 함.
    
    - General Domain:   Payment : 결제서비스로 3rd Party 외부 서비스를 사용하는 것이 경쟁력이 높음 (핑크색으로 이후 전환할 예정)



## 헥사고날 아키텍처 다이어그램 도출


![image](https://user-images.githubusercontent.com/67869000/98326344-5c11f100-2034-11eb-9cf4-6b435ab58e63.png)

    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함
    - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐


# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 8086이다)

```
cd order
mvn spring-boot:run

cd payment
mvn spring-boot:run

cd delivery
mvn spring-boot:run

cd statusview
mvn spring-boot:run

cd coupon
mvn spring-boot:run

cd review
mvn spring-boot:run

```

## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다: (예시는 review 마이크로 서비스). 이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용하려고 노력했다. 하지만, 일부 구현에 있어서 영문이 아닌 경우는 실행이 불가능한 경우가 있기 때문에 계속 사용할 방법은 아닌것 같다. (Maven pom.xml, Kafka의 topic id, FeignClient 의 서비스 id 등은 한글로 식별자를 사용하는 경우 오류가 발생하는 것을 확인하였다)

```
package pizzalgh;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;

@Entity
@Table(name="Review_table")
public class Review {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private Long addpizzaId;
    private String addorderstatus = "AddedOrdered";
    private Long addqty;
    private Long orderedId;
    private String reviewtext;

    @PostPersist
    public void onPostPersist(){
        Written written = new Written();
        BeanUtils.copyProperties(this, written);
        written.publishAfterCommit();


        Addordered addordered = new Addordered();
        BeanUtils.copyProperties(this, addordered);
        addordered.publishAfterCommit();

        //Following code causes dependency to external APIs
        // it is NOT A GOOD PRACTICE. instead, Event-Policy mapping is recommended.

        pizzalgh.external.Order order = new pizzalgh.external.Order();
        // mappings goes here

        if(this.getAddqty() != null && this.getAddqty() != 0 )
        {
            order.setOrderStatus("Ordered");
            order.setPizzaId(addordered.getAddpizzaId());
            order.setQty(addordered.getAddqty());

            ReviewApplication.applicationContext.getBean(pizzalgh.external.OrderService.class)
                    .order(order);

        }




    }


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public Long getAddpizzaId() {
        return addpizzaId;
    }

    public void setAddpizzaId(Long addpizzaId) {
        this.addpizzaId = addpizzaId;
    }
    public String getAddorderstatus() {
        return addorderstatus;
    }

    public void setAddorderstatus(String addorderstatus) {
        this.addorderstatus = addorderstatus;
    }
    public Long getAddqty() {
        return addqty;
    }

    public void setAddqty(Long addqty) {
        this.addqty = addqty;
    }
    public Long getOrderedId() {
        return orderedId;
    }

    public void setOrderedId(Long orderedId) {
        this.orderedId = orderedId;
    }
    public String getReviewtext() {
        return reviewtext;
    }

    public void setReviewtext(String reviewtext) {
        this.reviewtext = reviewtext;
    }
}


```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or In Memory DB) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package pizzalgh;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface ReviewRepository extends PagingAndSortingRepository<Review, Long>{

}
```
- 적용 후 REST API 의 테스트
```
# 리뷰 
http POST http://localhost:8086/reviews addpizzaId=2 addqty=2 reviewtext="review1"

```

![image](https://user-images.githubusercontent.com/67869000/98327357-abf1b780-2036-11eb-8d02-08d2e9cfafe8.png)


## 폴리글랏 퍼시스턴스

H2가 아닌 HSQL in-memory HSQL을 사용함

```
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.4.0</version>
    <scope>runtime</scope>
</dependency>
```

![image](https://user-images.githubusercontent.com/67869000/98327541-17d42000-2037-11eb-8cf2-12c62091ed23.png)



## 동기식 호출 과 Fallback 처리

분석단계에서의 조건 중 하나로 "재"주문(Addedorder)->주문(order) 간의 호출은 동기식 일관성을 유지하는 트랜잭션으로 처리하기로 하였다. 호출 프로토콜은 이미 앞서 Rest Repository 에 의해 노출되어있는 REST 서비스를 FeignClient 를 이용하여 호출하도록 한다. 

- 주문서비스를 호출하기 위하여 FeignClient를 이용하여 Service 대행 인터페이스 (Proxy) 를 구현 

```
# Review의 OrderService.java


package pizzalgh.external;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.Date;

//@FeignClient(name="order", url="http://order:8080")//origin
@FeignClient(name="order", url="http://localhost:8081")//local
//@FeignClient(name="order", url="http://api.url.order:8080")
public interface OrderService {

    @RequestMapping(method= RequestMethod.POST, path="/orders")
    public void order(@RequestBody Order order);

}
```

- "재"주문을 받은 직후(@PostPersist) 결제를 요청하도록 처리
```
# review.java (Entity)
   @PostPersist
    public void onPostPersist(){
        Written written = new Written();
        BeanUtils.copyProperties(this, written);
        written.publishAfterCommit();


        Addordered addordered = new Addordered();
        BeanUtils.copyProperties(this, addordered);
        addordered.publishAfterCommit();

        //Following code causes dependency to external APIs
        // it is NOT A GOOD PRACTICE. instead, Event-Policy mapping is recommended.

        pizzalgh.external.Order order = new pizzalgh.external.Order();
        // mappings goes here

        if(this.getAddqty() != null && this.getAddqty() != 0 )
        {
            order.setOrderStatus("Ordered");
            order.setPizzaId(addordered.getAddpizzaId());
            order.setQty(addordered.getAddqty());

            ReviewApplication.applicationContext.getBean(pizzalgh.external.OrderService.class)
                    .order(order);

        }
    }
```

- 동기식 호출에서는 호출 시간에 따른 타임 커플링이 발생하며, 결제 시스템이 장애가 나면 주문도 못받는다는 것을 확인:


```
# 주문 (order) 서비스를 잠시 내려놓음

#"재"주문처리
http http://localhost:8086/revies addpizzaId=3 reviewtext=“review3”   #Fail
```

![image](https://user-images.githubusercontent.com/67869000/98327825-d1cb8c00-2037-11eb-9a81-06294e8d083c.png)

```
#주문서비스 재기동
cd order
mvn spring-boot:run

#재주문처리
http http://localhost:8086/revies addpizzaId=3 reviewtext=“review3”   #Success
```
![image](https://user-images.githubusercontent.com/67869000/98327950-0b03fc00-2038-11eb-862e-91369d4ded51.png)

- 또한 과도한 요청시에 서비스 장애가 도미노 처럼 벌어질 수 있다. (서킷브레이커, 폴백 처리는 운영단계에서 설명한다.)




## 비동기식 호출 / 시간적 디커플링 / 장애격리 / 최종 (Eventual) 일관성 테스트

리뷰 작성이 이루어진 후에 쿠폰시스템으로 이를 알려주는 행위는 동기식이 아니라 비 동기식으로 처리하여 쿠폰 시스템의 처리를 위하여 주문이 블로킹 되지 않아도록 처리한다.
 
- 이를 위하여 리뷰작성에 기록을 남긴 후에 곧바로 쿠폰이 발행 되었다는 도메인 이벤트를 카프카로 송출한다(Publish)
 
```
@Entity
@Table(name="Review_table")
public class Review {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private Long addpizzaId;
    private String addorderstatus = "AddedOrdered";
    private Long addqty;
    private Long orderedId;
    private String reviewtext;

    @PostPersist
    public void onPostPersist(){
        Written written = new Written();
        BeanUtils.copyProperties(this, written);
        written.publishAfterCommit();


    }
```
- 쿠폰발행 이벤트에 대해서 이를 수신하여 자신의 정책을 처리하도록 PolicyHandler 를 구현한다:

```
package pizzalgh;

import pizzalgh.config.kafka.KafkaProcessor;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Service;

@Service
public class PolicyHandler{
    @Autowired
    CouponRepository couponRepository;

    @StreamListener(KafkaProcessor.INPUT)
    public void onStringEventListener(@Payload String eventString){

    }

    @StreamListener(KafkaProcessor.INPUT)
    public void wheneverWritten_PublishCoupon(@Payload Written written){

        if(written.isMe()){
            System.out.println("##### listener PublishCoupon : " + written.toJson());
            Coupon coupon = new Coupon();
            coupon.setOrderId(written.getOrderedId());


            couponRepository.save(coupon);
        }
    }

}


```

쿠폰 시스템은 배송서비스와 완전히 분리되어있으며, 이벤트 수신에 따라 처리되기 때문에, 쿠폰 시스템이 유지보수로 인해 잠시 내려간 상태라도 주문을 받는데 문제가 없다:

```
쿠폰 서비스를 잠시 내려놓음 
```

![image](https://user-images.githubusercontent.com/67869000/98328258-a72e0300-2038-11eb-895b-015853d87323.png)

```
쿠폰 서비스 재기동
cd coupon
mvn spring-boot:run

쿠폰이 발행됨을 확인
```
![image](https://user-images.githubusercontent.com/67869000/98328469-291e2c00-2039-11eb-87f6-63ff56deb9b3.png)


## CQRS 적용

추가된 reviewtext 확인 가능

![image](https://user-images.githubusercontent.com/67869000/98328287-b7de7900-2038-11eb-89f4-cd924b75c092.png)


## gateway 적용

application.yaml파일에 소스 적용


![image](https://user-images.githubusercontent.com/67869000/98328380-f1af7f80-2038-11eb-9d6f-d6cd5026ab35.png)

호출확인

![image](https://user-images.githubusercontent.com/67869000/98328312-c7f65880-2038-11eb-826c-57b3566368b0.png)

# 운영

## CI/CD 설정
각 구현체들은 각자의 source repository 에 구성되었고, 사용한 CI/CD 플랫폼은 Azure를 사용하였으며, pipeline build script 는 각 프로젝트 폴더 이하에 deployment.yml, service.yml 에 포함되었다.

![image](https://user-images.githubusercontent.com/67869000/98328602-769a9900-2039-11eb-8000-56a33d07699d.png)

![image](https://user-images.githubusercontent.com/67869000/98328644-8dd98680-2039-11eb-91b9-8301f0e59317.png)

![image](https://user-images.githubusercontent.com/67869000/98328693-a6e23780-2039-11eb-8561-bad4908a9111.png)

![image](https://user-images.githubusercontent.com/67869000/98328713-b1043600-2039-11eb-9da8-d838f826f356.png)


## 동기식 호출 / 서킷 브레이킹 / 장애격리

* 서킷 브레이킹 프레임워크의 선택: Spring FeignClient + Hystrix 옵션을 사용하여 구현함

시나리오는 재주문(Addorder)-->주문(order) 의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 결제 요청이 과도할 경우 CB 를 통하여 장애격리.

- 재주문(Addorder)의 application.yaml 파일에 Hystrix 를 설정: 요청처리 쓰레드에서 처리시간이 500 밀리가 넘어서기 시작하여 어느정도 유지되면 CB 회로가 닫히도록 (요청을 빠르게 실패처리, 차단) 설정
```
# application.yml
feign:
  hystrix:
    enabled: true

hystrix:
  command:
    default:
      execution.isolation.thread.timeoutInMilliseconds: 500

```
- 피호출 서비스 order onPostPersist영역의 부하코드 추가 - 400 밀리에서 증감 350 밀리 정도 왔다갔다 하게
```
# order.java (Entity)

    @PostPersist
    public void onPostPersist(){
        Paid paid = new Paid();
        BeanUtils.copyProperties(this, paid);
        paid.publishAfterCommit();

        try {
            Thread.sleep((long) (400 + Math.random() * 350));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

![image](https://user-images.githubusercontent.com/67869000/98332658-6c30cd00-2042-11eb-9925-24e97d296806.png)

-운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌. 
72% 가 성공하였고, 고객 사용성에 있어 좋지 않기 때문에 28%를 커버하기위하여  Retry 설정과 동적 Scale out (replica의 자동적 추가,HPA) 을 통하여 시스템을 확장 해주는 후속처리가 필요.


### 오토스케일 아웃

앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다. 


- 결제서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 20프로를 넘어서면 replica 를 10개까지 늘려준다:
```
kubectl autoscale deployment review --cpu-percent=20 --min=1 --max=10
```
![image](https://user-images.githubusercontent.com/70673848/98128510-5cf33780-1efb-11eb-8f1d-56e3eacb5d6a.png)

- CB 에서 했던 방식대로 워크로드를 2분 동안 걸어준다.
```
siege -c100 -t120S -r10 --content-type "application/json" 'http://review:8080/reviews POST {"pizzaId":1,"addqty":2}

```
- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다:
```
kubectl get deploy review -w
```
- 어느정도 시간이 흐른 후 (약 60초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:
![image](https://user-images.githubusercontent.com/67869000/98328900-31c33200-203a-11eb-8ce3-d7ae9dfbd322.png)


## 무정지 재배포

* 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함

- seige 로 배포작업 직전에 워크로드를 모니터링 함.
```
siege -c100 -t120S -r10 --content-type "application/json" 'http://review:8081/reviews POST {"pizzaId":10, "addqty":10}'

```

- 새버전으로의 배포 시작
```
kubectl set image ...
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인

![image](https://user-images.githubusercontent.com/67869000/98328987-76e76400-203a-11eb-9ff9-2d0f1d115c00.png)



배포기간중 Availability 가 평소 100%에서 95% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해 Readiness Probe 를 설정함:

```
# deployment.yaml 의 readiness probe 의 설정:

kubectl apply -f kubernetes/deployment.yaml
```

- 동일한 시나리오로 재배포 한 후 Availability 확인:

![image](https://user-images.githubusercontent.com/67869000/98329026-9088ab80-203a-11eb-9ce9-86df33d3c191.png)


배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.

## Liveness 구현

- review 의 depolyment.yaml 소스 설정
서비스포트 8080이 아닌 고의로 8081로 포트 변경하여 강제로 재기동 되도록 설정 한다.

```
비정상 상태의 pod 정보 확인
kubectl get pod review-76d87c986-w67xw
```

![image](https://user-images.githubusercontent.com/67869000/98329062-a72f0280-203a-11eb-9b61-9a46bc41d1d2.png)






