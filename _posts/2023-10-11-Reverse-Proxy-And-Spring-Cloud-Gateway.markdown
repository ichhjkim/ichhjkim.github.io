---
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: id_ko-2
title: Reverse Proxy와 Spring Cloud Gateway

# post specific
# if not specified, .name will be used from _data/owner/en.yml
author: ichhjkim
# multiple category is not supported
category: auto generated
# multiple tag entries are possible
tags: [spring, spring cloud, reverse proxy, spring cloud gateway, scg, SCG]
# thumbnail image for post
img: ":데이터_실시간_노출.png"
# disable comments on this page
comments_disable: false

# publish date
date: 2023-10-11 01:00:00 +0900

# seo
# if not specified, date will be used.
#meta_modify_date: 2023-10-09 23:16:04 +0900
# check the meta_common_description in _data/owner/ko.yml
# meta_description: "Server Sent Events(SSE)에 대해 알아보고 Java Spring, Javascript으로 구현해보는 게시글입니다."

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

Reverse Proxy에 대해 알아보고 Reverse Proxy의 일종인 Spring cloud gateway에 대해 알아봅니다.

## Proxy Server과 Reverse Proxy

프록시 서버는 간단히 말하면 클라이언트-서버 사이의 **중계자**로서 통신을 대리로 수행하는 프로그램입니다.
클라이언트가 프록시서버를 통해서 다른 네트워크 서비스에 간접적으로 접속 할 수 있게 되는 것이죠.

**간접적**이라는 단어에서 감이 오듯이 클라이언트의 접속을 제어할 수 있는 역할을 하기 때문에
성능, 보안성, 운영에서의 안전성 등을 증대 시킬 수 있습니다.

Proxy Server는 포워드 프록시와 리버스 프록시 서버로 나누어지는데요.
이 게시글에서는 리버스 프록시에 대해 주로 다루도록하겠습니다.

## Reverse Proxy(리버스 프록시)란?

![리버스_프록시_구조](../assets/img/posts/reverse_proxy.png)

리버스 프록시는 위의 그림 처럼 Internal Network내부에서 서버 앞에 놓여있는 것을 의미합니다.

클라이언트는 웹 서비스를 접근할 때 서버에 요청을 하는 것이 아니라 '프록시'로 요청을 하게 되고,
프록시가 서버에 요청해서 응답값을 클라이언트에 전달합니다.

이렇게 리버스 프록시를 사용하는 이유는 아래와 같은 이유들이 있는데요.

첫번째로는, **로드밸런싱(Load Balancing)** 을 위해 사용합니다.
'단일 서버'로는 대용량 트래픽을 감당하기 어렵습니다. 그래서 여러 대 서버를 두고 요청을 분산시키게 되는데요.
요청을 분산시킬 때 사용하는 것이 '로드 밸런싱'입니다.

리버스 프록시는 여러 대의 서버앞에 위치해서 서버 요청을 분산시킬 수 있기 때문에
서버 부하를 분산시킬 수 있습니다.

두번째로는, **보안**적인 측면에서 장점이 있습니다. 리버스 프록시를 사용하면 본래의 서버 IP주소가 아니라 Proxy서버 주소가 노출되기 때문에,
서버에 대한 공격을 막는데 유용합니다.

세번째로는, **성능** 향상에 도움이 됩니다.
리버스 프록시에는 성능 향상을 위해 캐시데이터를 저장할 수 있기 때문에 캐싱된 데이터를 바탕으로  더 빠른 결과값을 얻을 수 있습니다.


## Spring Cloud Gateway란?

`Spring Cloud Gateway(SCG)`는 API Gateway로서 사용자 요청에 따라 적절한 마이크로 서비스에 라우팅하는 서버인데요.
보안/모니터링 등의 기능이 더 추가된 일종의 `Reverse Proxy`로 많이 사용되고 있습니다.

저희 팀내에서도 Spring Cloud Gateway를 Reverse Proxy로서 사용하고 있는데요,
앞단에서 라우트 제어/로드밸런싱/인증인가 처리가 가능하고 Spring을 이용한 구현이 자유로워서
활발하게 사용하고 있습니다.

### 동작방식

Spring Cloud Gateway는 다른 대부분의 Spring과는 다르게 Tomcat이 아닌 Netty를 사용합니다.
Proxy서버는 모든 클라이언트의 요청이 통과하는 곳이이게 그 어느 서버보다 성능적인 측면이 중요하기 때문인데요.

Netty는 비동기 WAS이고 1 Tread: Many Requests 방식이기 때문에 1 Tread: 1 Request 방식인 Spring보다 더 많은 요청을 처리할 수 있습니다.

![Spring_Cloud_Gateway_동작방식](../assets/img/posts/Spring-Cloud-Gateway.png)

[출처:스프링 클라우드 게이트 웨이 공식문서](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html) {:data-align="center"}

## Spring Cloud Gateway의 3가지 구성요소

### Route
Route는 요청으로 들어온 URI의 조건이 True인 경우(Predicate에서 조건 확인), 매핑된 경로로 매칭시켜줍니다.
항목을 식별하기 위한 ID, 목적지 URI, Predicate, Filter로 구성되어 있습니다.

### Predicate
주어진 요청이 주어진 조건을 충족하는지 확인하는 조건문 요소입니다.
조건에 매핑되지 않으면 Http 404 Not Found를 응답합니다.

### Filter

Spring Cloud Gateway로 들어오는 요청과 응답에 대해 수정을 가능하게 해줍니다.
예를 들어, CircuitBreaker, PrefixPath, redis-rate-limiter,
AddRequestHeader등이 있습니다.

아래 yaml과 같이 코드는 작성할 수 있는데요.
path /order/**로 요청이 오면 uri http://localhost:8080으로 요청을 보냅니다.
이때 AuthFilter에서 인증인가를 체크합니다.

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service1
          uri: http://localhost:8080
          predicates:
            - Path=/order/**
          filters:
            - name: AuthFilter
              args:
                baseMessage: authFilter
                preLogger: true
                postLogger: true
```

<!-- outline-end -->
