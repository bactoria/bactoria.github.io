---
layout: post
title:  "Same-Origin Policy와 해결 방안"
categories: Rest
author: bactoria
---

Vue.js 프로젝트에서 Rest API로 Ajax요청을 했더니 다음과 같은 에러를 직면했다.

&nbsp;

**Chrome**  
![image](https://user-images.githubusercontent.com/25674959/57191428-81660080-6f60-11e9-87f4-2533cf1cb640.png)

&nbsp;

**Firefox**  
![image](https://user-images.githubusercontent.com/25674959/57191423-71e6b780-6f60-11e9-8ec7-172f7974d606.png)

&nbsp;

**해결법**  
1. CORS 허용 할 메소드에 @CrossOrigin를 추가
2. 시큐리티 설정에 Options Method를 permitAll

&nbsp;

REST API 프로젝트를 2개를 했었는데, 2번 모두 SOP 문제에 직면했었고 검색하면서 해결했다. 그래서 지금은 검색하지 않아도 해결 할 수 있다. 
하지만 SOP가 무엇이고 왜 나온 것인지, @CrossOrigin을 적용하면 구체적으로 무슨 일이 이루어지는지, Options 메서드를 허용 해야 하는 이유에 대해서는 깊이 생각을 하지 않았던 것 같다.

해결법은 아는데, 문제가 생긴 근본적인 원인을 모르는 것이다.

프로그래머는 프로덕션을 만드는 사람이고, 서비스를 정상적으로 되게 하는 사람이고, 서비스가 잘 돌아가게 하는 사람이라고 많이 들었다.

하지만 최소한 왜 그런건지는 이해를 해야할 것 같다. 어떻게 보면 원인을 알아야 생산성이 더 높아진다고 보는게 맞는 것 같기도 하네. 막무가내로 해결하는 것은 대학교에서 시험치기 위해 암기하는거랑 다를게 없어보인다.

&nbsp;

## Ajax 로 부터

Ajax부터 시작하면 될것 같다.

이전 웹페이지들은 새로운 내용을 갱신하기 위해 페이지를 리프레시 했었는데,

페이지를 리프레시 하지 않고도 서버와 데이터를 주고 받을 수 있게 되었다. 이걸 Ajax라고 한다.

[지메일이 핫메일을 이긴 진짜 이유 - 조성문님](https://sungmooncho.com/2012/12/04/gmail-and-ajax/2004년)에 따르면 2004년 4월 1일에 Gmail이 나왔는데, 메일을 확인하거나 발송할 때 화면 전체가 깜빡이는 것이 없어서 핫메일보다 빠르고 쾌적했다고 한다. Ajax 를 사용했기 때문이다. (Gmail의 성공 요인은 Ajax가 전부는 아니라고 한다. 어쨌든 빠른건 사실이다.)

이 Ajax는 `XMLHttpRequest` 객체를 통해 구현할 수 있으며, 여기에는 보안 이슈가 있었디.

&nbsp;

## CSRF 공격 (Cross Site Request Forgery)

만약 내가 웹페이지를 만들었는데, 아래와 같은 스크립트를 넣어서, 접속자들의 페이스북 타임라인 글들을 삭제하는 것이 가능하다면 ?

**http://fakebook.com** 
```javascript
var url = "http://facebook.com/timelines"; //url은 막무가내로 만든거
var xhr = new XMLHttpRequest();
xhr.open("DELETE", url, true);
xhr.onload = function () {
    var users = JSON.parse(xhr.responseText);
    if (xhr.readyState == 4 && xhr.status == "200") {
        console.log("삭제 완료");
    } else {
        console.error("앗 실패했군");
    }
}
xhr.send(null);
```

이전에 facebook에 로그인을 한 상태에서 fakebook에 방문한다면 그 사람의 타임라인 글을 삭제해버릴 수 있다. (물론 페이스북을 저렇게 단순하게 공격할 수도 없겠지만..)

&nbsp;

## SOP (Same-Origin Policy)

SOP는 웹 사이트 방문 시에 다른 Origin의 DOM에 접근시 허용하지 않는 정책이었다. 유저가 신뢰할 수 없는 사이트 방문에 대한 걱정을 덜 수 있다.

Ajax가 나온 이후 CSRF를 막기 위해 XMLHttpRequest에도 확장 적용되었다. 웹 브라우저는 현재 웹 페이지에 포함된 스크립트가 어떤 웹 페이지의 데이터에 액세스하는 것을 허용하기 위해서는, 이 두 웹 페이지가 `Same Origin`이어야 한다.

즉, fakebook을 방문했을 때, facebook으로 스크립트를 보낼 수 없게끔 웹 브라우저가 막는 것이 가능해진다.

&nbsp;

### Same Origin 이란?

**Same Origin** : 두 URL의 `스키마`, `호스트`, `포트`가 같을 경우

**Cross Origin** : 3개 중에도 하나라도 다를 경우

&nbsp;

- `http://test.com:3000`  & `https://test.com:3000` -> Cross Origin
- `http://test.com:3000`  & `http://test.com:8080` -> Cross Origin
- `http://test.com:3000`  & `http://real.com:3000` -> Cross Origin

&nbsp;

Same Origin이 아닐 경우(Cross Origin 일 경우), **브라우저에 의해 요청이 거절된다.**

거절된 것이 이 글의 가장 위에 있는 이미지이다.

&nbsp;

## 해결법

### 1. JSONP

2005년 제안된 방법.

JSONP를 사용하면 XMLHttpRequest 객체를 사용하지 않고 Cross Origin인 서버로부터 JSON 데이터를 받을 수 있다.

XMLHttpRequest 객체를 사용하지 않기 때문에 SOP를 위배하지 않는 것이다.

JSONP는 서버로부터 콜백함수를 실행하는 코드를 받아온다.

**요청**
```javascript
// ...
<script src="http://food-truck.shop/search?callback=loadTruckList&s=떡볶이"></script>
```

`http://food-truck.shop/search?`**`callback=loadTruckList`**`&s=떡볶이` 를 아래와 같이 바꿔준다.

**응답**
```javascript
// ...
loadTruckList(["John Miller","John C. Potter"])
```

food-truck.shop에서 위와 같이 변환해준 것이다.

jsonp는 클라이언트에서 쓰겠다고 해서 쓸 수 있는 것이 아니라, 서버에서 위의 형식처럼 함수로 응답해주어야 한다.

즉, 서버에서 json 데이터를 함수로 wrapping해서 응답해줘야 하는데 이처럼 감싸는 행위를 Padding이라고 불러서 JSONP 라고 부르는 것 같다.

GET Method만을 지원한다는 단점과 보안상의 이슈로 인해 CORS를 추천하는 분위기 라고 함.

&nbsp;

### 2. CORS (Cross-Origin Resource Sharing)

- 2004년 3월 VoiceXML 브라우저의 안전한 Cross-Origin Data 요청을 허용하기 위해 Cross-Origin 지원을 제안함
- 2006년 5월 최초의 W3C 작업 초안이 제출됨
- 2009년 3월 초안은 CORS(Cross-Origin Resource Sharing)로 개명됨
- 2014년 1월 W3C 권고안으로 승인([W3C Recommendation of CORS](https://www.w3.org/TR/cors/)) (구식)
- 201?년 https://fetch.spec.whatwg.org/ (현재)

&nbsp;

JSONP 패턴의 현대적인 대안.

서버와 클라이언트가 정해진 헤더를 통해 서로 요청이나 응답에 반응할지 결정하는 방식으로 CORS라는 이름으로 표준화됨.

![cors_xhr](https://user-images.githubusercontent.com/25674959/57578581-fd64c900-74c9-11e9-8096-f1c6d153c421.png)

&nbsp;

서버에 
클라이언트가 서버로 HTTP Request를 보낼 때 
- GET ::  
  - custom header가 없으면
- HEAD ::  
  - custom header가 없으면
- POST ::
  - ContentType이 standard(`application/x-www-form-urlencoded` or `multipart/form-data` or `text/plain`) 이고, custom header가 없으면

Cross-Origin에 리소스를 요청 할 수 있다.

그 밖의 경우 (ex. POST로 요청에 Content-type이 application/json 인 경우 -> custom header를 가짐.)에는 서버에 **Preflight Request**를 보내야 함.

### Preflight Request

클라이언트가 서버로 HTTP Request를 보낼 때 위의 기준을 충족하지 못하면, **브라우저가 자동으로 원래 요청보다 먼저 OPTIONS Method로 HTTP 요청하여, 원래 요청을 전송해도 안전한지 확인한다.** 

위 그림에서 빨간 부분이 Preflight Request 과정이다.

```javascript
// Request Message
OPTIONS /users/:id 
Access-Control-Request-Method: DELETE 
Access-Control-Request-Headers: origin, x-requested-with
Origin: https://example.com

// Response Message
HTTP/1.1 200 OK
Content-Length: 0
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, GET, OPTIONS, DELETE
```

&nbsp;

### JSONP vs CORS

JSONP는 GET Method만 지원하는 반면, CORS는 다른 Method도 지원한다. 

JSONP는 사이트 간 스크립팅(XSS) 문제를 야기할 수 있다.

CORS를 사용하면 JSONP보다 더 나은 오류 처리를 지원하는 XHR(XMLHttpRequest)를 사용할 수 있다.

CORS는 XHR Level2에서 적용되었는데, XHR2는 대부분의 최신 웹 브라우저에서 지원한다.

![image](https://user-images.githubusercontent.com/25674959/57901997-3f16ba80-78a2-11e9-9aa9-f716a80a582f.png)
(https://caniuse.com/#feat=xhr2)

IE10 이하는 CORS 지원하지 않음.
IE9, 8 이하에서는 `XDomainRequest`를 사용해야 하고, 그 이하는 JSONP를 사용해야 한다. JSONP는 CORS가 지원되기 이전의 기존 브라우저에서도 동작하기에 브라우저 호환을 중요시하는 곳에서는 JSONP도 쓰고 있을 것 같다.

(`XDomainRequest`가 있지만, Get, Post만 지원함. => TODO :: CRUD 어떻게?) 

```javascript
// XDomainRequest

var xdr = new XDomainRequest();
xdr.open("get", "http://www.contoso.com/xdr.aspx");
xdr.send();
```

Axios의 경우 0.6.0 버전에서는 XDomainRequest로 IE8을 지원했음. (현재 Axios는 IE11만 지원)

&nbsp;

## 참고

- https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/cors.html
- https://tools.ietf.org/html/rfc6749#section-4.3.2
- https://brunch.co.kr/@adrenalinee31/1
- https://heowc.dev/2018/03/13/spring-boot-cors/
- https://www.zerocho.com/category/HTTP/post/5b4c4e3efc5052001b4f519b
- https://wit.nts-corp.com/2014/04/22/1400
- https://itstory.tk/entry/CSRF-%EA%B3%B5%EA%B2%A9%EC%9D%B4%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-CSRF-%EB%B0%A9%EC%96%B4-%EB%B0%A9%EB%B2%95
- https://greatkim91.tistory.com/14
- https://www.w3.org/Security/wiki/Same_Origin_Policy
- SOP
    + https://www.internetmap.kr/entry/Sameorigin-policy
- JSONP
    + https://ko.wikipedia.org/wiki/JSONP
    + https://kingbbode.tistory.com/26
    + http://charlie0301.blogspot.com/2013/12/jquery-ajax-same-origin-policy-jsonp.html
    + http://dev.epiloum.net/1311
- CORS
    + https://dev.to/effingkay/cors-preflighted-requests--options-method-3024
    + https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS
    + [CORS 설정해도 IE에서 크로스 도메인간 요청 불가 원인](http://www.codejs.co.kr/cors-%EC%84%A4%EC%A0%95%ED%95%B4%EB%8F%84-ie%EC%97%90%EC%84%9C-%ED%81%AC%EB%A1%9C%EC%8A%A4-%EB%8F%84%EB%A9%94%EC%9D%B8%EA%B0%84-%EC%9A%94%EC%B2%AD-%EB%B6%88%EA%B0%80-%EC%9B%90%EC%9D%B8-no-transport/)




