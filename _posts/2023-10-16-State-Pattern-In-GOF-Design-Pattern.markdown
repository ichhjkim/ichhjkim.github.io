---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: id_ko-4
title: 스테이트 패턴(State Design Pattern) in Java

# post specific
# if not specified, .name will be used from _data/owner/ko.yml
author: KIM HYUN JI
# multiple category is not supported
category: auto generated
# multiple tag entries are possible
tags: [state pattern, 스테이트 패턴, 디자인 패턴, design pattern, GoF, 상태패턴]
# thumbnail image for post
img: ":circuit_breaker.png"
# disable comments on this page
comments_disable: false

# publish date
date: 2023-10-17 17:00:00 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: 2023-10-17 17:00:00 +0900
# check the meta_common_description in _data/owner/ko.yml
# meta_description: "GoF Design Pattern 중 State Pattern에 대해 알아보고 적용해보는 게시글입니다."

# optional
# please use the "image_viewer_on" below to enable image viewer for individual pages or posts (_posts/ or en/_posts folders).
# image viewer can be enabled or disabled for all posts using the "image_viewer_posts: true" setting in _data/conf/main.yml.
#image_viewer_on: true
# please use the "image_lazy_loader_on" below to enable image lazy loader for individual pages or posts (_posts/ or en/_posts folders).
# image lazy loader can be enabled or disabled for all posts using the "image_lazy_loader_posts: true" setting in _data/conf/main.yml.
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
# published: true

---

<!-- outline-start -->

오늘은 Spring Cloud Circuir Breaker에 대해 알아보겠습니다.

MSA로 개발을 진행하다 보면 필연적으로 여러 도메인 서버들과 응답을 주고 받게 됩니다.

이때, 한 도메인이 전혀 응답하지 않게 되는 상황이 발생하면 어떻게 될까요?



동작하지 않는 도메인을 계속 호출하고 응답을 대기하다 보면 다른 의존성 있는 도메인까지 장애가 확산될 수 있습니다.

이런 연쇄적인 장애 상황은 전체 서비스에 영향을 주기 때문에 반드시 막아야 합니다.



그러기 위해서는 장애가 발생한 도메인 서버에 대한 접속 차단이 필요합니다.

이를 자동화한 것이 바로 Circuit Breaker입니다!

## Circuit Breaker란?

타 서버 접속의 접속 실패율이 임계치를 넘어섰을 때, Circuit을 오픈하여 장애가 다른 서비스로 확산되지 못하도록 미리 정의해둔 Fallback Response를 내보내는 패턴입니다.


![Circuit Breaker](../assets/img/posts/circuit_breaker.png)

**OPEN**: 에러율이 임계치를 넘으면 OPEN상태가 되며 모든 접속이 차단됩니다. (실제 요청을 날리지 않고 에러를 바로 발생시키는 방식)

**CLOSE**:  초기 상태, 모든 접속이 정상적으로 실행됩니다.

**HALF_OPEN**: OPEN 상태 중간에 일정시간 마다 한번씩 요청을 다시 날려 서버 응답값이 성공인지 확인합니다. 이따 성공이면 CLOSE, 실패이면 OPEN으로 되돌아 갑니다.


## OpenFeign의 Circuit Breaker 적용

저희 팀내에서는 서버간 통신을 할때 주로 Spring Cloud OpenFeign을 사용하고 있는데요
Spring cloud OpenFeign은 HystrixFeign이라는 Hystrix기반 Circuit Breaker을 지원하고 있기 때문에
Circuit Breaker도 OpenFeign을 통해서 간편하게 적용하고 있습니다.


(최근 hytrix는 더이상 유지관리 모드로 개발되고 있지 않기 때문에 Resilience4j가 대안으로 권장되고 있습니다.
추후 OpenFeign도 Resilience4j를 지원할 예정이라고 합니다)



### 적용 방법

OpenFeign의 Circuit Breaker 적용은 기본적으로 OpenFeign이 소스 내 적용되어 있다고 가정하고 설명드리겠습니다.

#### 1) application.yaml

```yaml
spring:
  cloud:
    openfeign:
      circuitbreaker:
        enabled: true # 1) circuitbreaker 사용 설정

resilience4j:
  circuitbreaker:
    configs:
      default:
        sliding-window-type: COUNT_BASED # 2) 최근 호출 횟수를 기반으로 Circuit Breaker의 상태가 결정
        slidingWindowSize: 100 # 3) sliding window 크기 (default)
        minimumNumberOfCalls: 100 # 4) failureRate, slowCallRate 를 계산하기 위한 최소 call 갯수 (default)
        waitDurationInOpenState: 30s # 5) OPEN 상태에서 HALF-OPEN 상태로 변경 대시 시간
        failureRateThreshold: 30 # 6) 실패율 임계값 (임계값 초과시 CLOSED -> OPEN 으로 변경, default : 50)
```
<br>

* **sliding-window-type** : 슬라이딩 윈도우의 유형을 지정합니다. COUNT_BASED로 설정하면 호출 횟수를 기반으로 슬라이딩 윈도우가 동작합니다. 다른 옵션으로 TIME_BASED를 사용할 수도 있습니다.
* **slidingWindowSize** : 슬라이딩 윈도우의 크기를 지정
* **minimumNumberOfCalls**: 실패율과 느린 호출 비율을 계산하기 위한 최소 호출 횟수를 설정합니다. 이 횟수보다 적은 호출이 있을 경우 Circuit Breaker는 상태를 변경하지 않습니다. 위의 예에서는 최소 100번의 호출을 설정하였으므로, 100번 미만의 호출이 있을 경우 Circuit Breaker의 상태가 변경되지 않습니다.
* **waitDurationInOpenState**: Circuit Breaker가 OPEN 상태에서 HALF-OPEN 상태로 전환될 때까지 기다리는 시간을 설정합니다. OPEN 상태에서 HALF-OPEN 상태로 전환되어 서비스 호출을 테스트합니다.
* **failureRateThreshold**: 실패율 임계값을 설정합니다. 실패율 임계값은 Circuit Breaker의 동작을 제어합니다. 실패율은 호출 중에 발생한 실패의 비율을 나타냅니다. 위의 예에서는 실패율 임계값을 30%로 설정하였으므로, 실패율이 30% 이상일 경우 Circuit Breaker가 CLOSED 상태에서 OPEN 상태로 변경됩니다.

<br>

> ###sliding-window란?
>슬라이딩 윈도우(Sliding Window)는 통계 수집 및 이벤트 추적을 위해 일련의 호출을 관찰하는 메커니즘입니다. 이것은 주로 Circuit Breaker와 같은 장애 처리 시스템에서 사용되며, 호출 통계 정보를 수집하여 시스템의 상태를 모니터링하고 장애를 검출하는 데 사용됩니다.


#### 2) OpenFeignConfig.class

```java
@Configuration
public class OpenFeignConfiguration {
    // feignClient 이름으로 패턴화
    @Bean
    public CircuitBreakerNameResolver circuitBreakerNameResolver() {
        return (String feignClientName, Target<?> target, Method method) -> feignClientName + "_" + method.getName();
    }
}
```

#### 3) Fallback적용-Exception 발생 시 실행되는 프로세스

##### 고객 정보를 불러오는 CustomerClient.class

```java
@FeignClient(
	name="Customer",
    url="http://localhost:8080/customer",
    configuration = OpenFeignConfig.class,
    fallbackFactory=CustomerFeignClientFallbackFactory.class
    )
public interface CustomerClient {
    @GetMapping(GET_CUSTOMER_INFO_URL)
    CustomerInfo getCustomer(@RequestHeader(HEADER_CUSTOMER_ID) String customerId);
}
```

##### 고객 도메인 서버 장애시 동작하는 FallbackFactory.class

```java
@Slf4j
@Component
public class CustomerFeignClientFallbackFactory implements FallbackFactory<CustomerFeignClient>{
  @Override
  public CustomerFeignClient create(Throwable cause) {
  	log.error("error type", cause);
    return new CustomerFeignClient() {
      @Override
      public CustomerInfo getCustomer(String customerId) {
        log.error("Fallback Occurred customerId={} cannot access", customerId);
        // 운영담당자 메일, 알림 보내는 로직 추가
        return new CustomerInfo();
      }
    };
  }
}
```

## 마무리
Spring Cloud OpenFeign를 통해 Circuit Breaker을 적용해보았습니다. 위에 작성한 내용 이외에도 OpenFeign은 여러 기능을 제공하는 아주 유용하고 편리하는 라이브러리입니다. 공식 문서에서 여러 기능을 적용해보세요. MSA에 활용하기에 유용한 기능들이 많습니다 😍


<!-- outline-end -->
