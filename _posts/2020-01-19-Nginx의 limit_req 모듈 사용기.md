---
layout: post
title: "Nginx의 limit_req 모듈 사용기"
date: 2020-01-19 20:11:31
categories: Deploy
tags: nginx limit_req
author: bactoria
---

### 이슈 발생

두 달전에 있었던 일이다. 프로젝트의 발표를 위해 서버를 로컬에 배포해 두었고, 발표 도중에 서버가 터지는 이슈가 발생했다.  

발표가 끝난 후 노트북을 보니, 아래처럼 로그가 계속 찍혀 들어오고 있었다.  

누군가 내 서버에 Request Message를 매우 빠르게 보내고 있었고, 이에 내 서버는 다른 사람들의 요청을 처리하지 못한 것이다.

![image](https://user-images.githubusercontent.com/25674959/70389749-e9089080-1a06-11ea-80db-e957a3e6cfcc.png)  

(누군가 장난으로 트래픽을 보내고 있었다.)

&nbsp;

### 어떻게 공격했을까 ?

트래픽을 일으킨 친구에게 어떻게 했는지 물어보았고, 방법은 간단했다.

![image](https://user-images.githubusercontent.com/25674959/72674702-155f0480-3abd-11ea-9671-2b5b0883bb0d.png)

추천 버튼을 누르면, 서버로 request를 보내는데, 이 버튼을 이용하여 서버로 트래픽을 발생시켰다.

&nbsp;

![image](https://user-images.githubusercontent.com/25674959/70493580-8d93eb00-1b4b-11ea-80cc-6a22f45235b6.png)

개발자 도구를 이용하여 추천 버튼에 `attack`이라는 클래스를 추가한 후, 아래와 같은 스크립트를 이용하면 된다.

&nbsp;

![image](https://user-images.githubusercontent.com/25674959/70494734-ca61e100-1b4f-11ea-9afa-decb05084df2.png)

&nbsp;  

### 해결하는 방법

이를 해결하는 방법을 찾아보니 
1. 자바 애플리케이션 단에서 해결하는 방법이 있고
   - Bucket4J, RateLimitJ, RateLimiter
2. 웹서버(NGINX)에서 해결하는 방법 도 있었다.
   - limit_req

이 중 나는 nginx에서 limit_req 모듈을 이용하여 해결하는 방법을 찾아봤다.

&nbsp;

&nbsp;

## limit_req in Nginx

나는 NGINX를 HTTP, HTTPS 포트로 들어오는 요청을 서버 포트로 리다이렉트시키는 용도로만 쓰고 있다.

기존의 코드는 다음과 같다.

&nbsp;

**/etc/nginx/nginx.conf**  
```nginx
# ...

http {
    # ...

    server {
        listen       80;
        server_name  localhost;

        location / {
            proxy_pass http://localhost:4000;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
        }
        
        # ...
    }

}
```

여기서 request message 를 제한할 수 있는, limit_req 를 입히면 다음과 같다.

&nbsp;

**/etc/nginx/nginx.conf**  

```nginx
http {
    # ...

    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s; ### 1)

    server {
        listen       80;
        server_name  localhost;

        limit_req zone=one burst=5; ### 2)
        
        location / {
            proxy_pass http://localhost:4000;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            
            limit_req_status        429; ### 3)
            limit_req_log_level     error; ### 4)
       }

       # ...
    }
}
```

**1) `limit_req_zone $binary_remote_addr zone=myName:10m rate=5r/s;`**
   - `limit_req_zone` `키` `공유 메모리 영역` `rate` - zone을 생성함
     - `키` : 요청에 limit을 걸 클라이언트 기준
       - $remote_addr: 클라이언트의 IP 주소
       - **$binary_remote_addr**: 클라이언트 IP 주소 (이진 표현)
         - 약 16,000 개의 IPv4에 상태 정보를 1MB에 넣을 수 있음 (64bit platform 기준) 
     - `공유 메모리 영역` : 키의 상태를 유지하는 영역의 이름과 크기
       - **zone=myName:10m**: 공유 메모리 영역 크기는 10mb, 이름은 myName
       - **$binary_remote_addr** (클라이언트 IP 주소의 상태) 가 보관됨
     - `rate` : 요청 비율 제한 ( 초당요청(r/s), 분당요청(r/m) )
       - **rate=1r/m** : 1분에 1개씩 request를 Server로 보냄
       - **rate=12r/m** : 5초에 1개씩 request를 Server로 보냄  (12request/60초 => 5초에 1request)

**2) `limit_req zone=one burst=5;`**
   - `limit_req` `zone 이름` `burst`
     - `zone 이름`: 위에서 만든 zone을 가져다 씀.
     - `burst`: 일종의 버킷이다.
       - burst 용량을 초과하는 요청이 들어올 경우 429(Too Many Request) 에러 반환
   - `burst=5`인 위의 경우에 7개의 요청이 동시에 들어온다면 ?
     - `Server(1개), burst(5개), Error(1개)`
   - `burst=5`인 위의 경우에 6개의 요청이 동시에 들어온다면 ?
     - `Server(1개), burst(5개)`
   - `burst=4`인 위의 경우에 9개의 요청이 동시에 들어온다면 ?
     - `Server(1개), burst(4개), Error(4개)`

위의 예시는 절대적인 것은 아니다. 위의 예시들은 `burst`가 가득 차기 이전에 `burst` -> `Server`로 첫 요청을 먼저 수행한다고 가정한 것. 이에 `Server(1개)` 가 담겨있는 것임.

**3) `limit_req_status 429;`**
   - 이 설정을 별도로 하지 않으면 503 에러 응답
   - 이 설정을 설정하면 `burst`가 가득찼을 경우 429 에러 응답

&nbsp;

### 예시

**rate=12r/m, burst=5 인 Nginx에**

**한 클라이언트의 Request Message가 동시에 10개 들어온 경우**
- Ok(6개), Error(4개)
  - Server(1개)
  - burst(5개)
    - burst에 쌓인 Request를 5초당 1개 씩  Server로 보냄
  - Error(4개)
    - burst가 가득 찼으므로 에러 응답
- 최종 결과
  - OK (1번째 Request)
    - Error (7번째 Request)
    - Error (8번째 Request)
    - Error (9번째 Request)
    - Error (10번째 Request)
  - OK (2번째 Request)
  - OK (3번째 Request)
  - OK (4번째 Request)
  - OK (5번째 Request)
  - OK (6번째 Request)


**한 클라이언트의 Request Message가 동시에 10개 들어오고,**  
**7초 후에 Request Message가 동시에 2개 들어온 경우**
- Ok(7개), Error(5개)
  - (0초)
    - Server(1개)
    - burst(5개)
      - burst에 쌓인 Request를 5초당 1개 씩  Server로 보냄
    - Error(4개)
      - burst가 가득 찼으므로 에러 응답
  - (7초)
    - burst(1개)
      - 기존에 burst에는 4개의 Request가 대기중. 남은 공간 1개를 채움.
    - Error(1개)
      - burst가 가득 찼으므로 에러 응답
  - 최종 결과
    - OK (1번째 Request)
      - Error (7번째 Request)
      - Error (8번째 Request)
      - Error (9번째 Request)
      - Error (10번째 Request)
    - OK (2번째 Request)
      - Error (12번째 Request)
    - OK (3번째 Request)
    - OK (4번째 Request)
    - OK (5번째 Request)
    - OK (6번째 Request)
    - OK (11번째 Request)

(서버의 요청 처리 속도에 따라 다를 수 있다. ex. `1번째 Request`는 `10번째 Request` 뒤에 찍히는 경우 )

&nbsp;  

&nbsp;

## limit과 burst 차이

나는 초반에 `limit`과 `burst`가 헷갈려서 이해하는데 많은 시간을 소모했다.

`limit`은 Nginx -> Server 로 클라이언트 요청을 몇 초마다 보낼지를 나타내고,  
`burst`는 Nginx -> Server 로 보낼 클라이언트 요청을 잠시 담아두는 곳이다.

결국 `burst`에 담겨 있는 request들이 일정 시간(limit) 마다 서버로 요청된다. `burst`가 가득찼는데도 request가 들어오면 바로 Nginx가 에러로 응답하는 것이다.

&nbsp;

내가 정리한 것은 이까지고, 이 외에도 [공식 문서](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-http/
)에는 많은 정보들이 있다.

필요하면 다음에 다시 찾아봐야겠다.

&nbsp;

&nbsp;

## Test

누구든지 로컬에서 돌려볼 수 있도록 [https://github.com/bactoria/nginx-limit-req-test](https://github.com/bactoria/nginx-limit-req-test) 에 공유 해두었다.

![](https://user-images.githubusercontent.com/25674959/72679251-cdf26b80-3af0-11ea-824e-fbe94d8a6aa5.png)
