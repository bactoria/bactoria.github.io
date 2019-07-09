---
layout: post
title:  "Unable to acquire JDBC Connection"
categories: Deploy
author: bactoria
---

블로그좀 들어가보려고 했더니 죽었다...

![default](https://user-images.githubusercontent.com/25674959/53799766-80275000-3f7e-11e9-89e1-7c4c7e80152a.PNG)

&nbsp;

당연히 Nuxt 문제인 줄 알았는데 Server 문제였다.

&nbsp;

서버에 **/api/posts** 쿼리를 날려보니 다음과 같은 응답이 왔다.

```json
{   
    "timestamp":"2019-03-05T09:57:45.768+0000",
    "status":500,
    "error":"Internal Server Error",
    "message":"Could not open JPA EntityManager for transaction; nested exception is org.hibernate.exception.JDBCConnectionException: Unable to acquire JDBC Connection",
    "path":"/api/posts"
}
```

&nbsp;

DB 연결 실패..

[우아한 개발자 블로그](http://woowabros.github.io/experience/2017/01/20/billing-event.html) 에서 비슷한 이슈가 있었는데,

해결한 것을 보니.. 고생좀 하겠다 라고 생각하면서

서버 EC2에서 DB로 psql 쿼리를 날렸다.

-> 접속이 안된다.

&nbsp;
&nbsp;

뭔가 알 것 같았다.

어제 RDS의 인바운드 그룹을 건드렸는데 실수로 EC2의 보안그룹을 지운 것이다.

&nbsp;

해야할게 생겼다.

1. 현재 Server 문제인데 Nuxt도 에러를 뱉어버린다. 서버가 문제일 경우 별도의 페이지를 만드는게 좋을 것 같다.

2. 정상적으로 동작하지 않을 때 Noti가 와야 한다.

3. EC2 보안그룹을 추가하는 대신, EC2의 Elastic IP를 추가하면 왜 안되는가?




