---
layout: post
title:  "final은 immutable한가 ?"
categories: Java
author: bactoria
---

> final은 키워드이고, Immutable은 패턴이다.

final은 Immutable인가?

아니다

&nbsp;

**final**
변수의 참조주소는 변경될 수 없다.

&nbsp;

**immutable**
객체내부의 값들이 수정될 수 없다.

&nbsp;

String.class 는 immutable이다.  

String 객체는 한번 선언되면 그 값이 변경되지 않기 때문

&nbsp;
&nbsp;

### Example

```Java
String name = "Bactoria";
name = "Junoh";
```

"Bactoria"객체의 value가 "Junoh"로 변경된 것 처럼 보일 수도 있지만,

아래처럼 이루어진다.

![image](https://user-images.githubusercontent.com/25674959/52163252-e4a68380-2722-11e9-98db-dea1e748a078.png)

![image](https://user-images.githubusercontent.com/25674959/52163263-030c7f00-2723-11e9-8782-4f9e96433ea3.png)

&nbsp;

`final`은 참조를 변경하지 못한다.

```java
final Stirng name = "Bactoria";
name = "Junoh";
```

![image](https://user-images.githubusercontent.com/25674959/52163297-744c3200-2723-11e9-94ab-0aeb7a66e228.png)

이펙티브자바에서 `변경 불가능 클래스`는 String 과 같은 immutable class를 말한다.

final이랑 헷갈리지말자.

&nbsp;
&nbsp;

## 참고자료

[What is the difference between immutable and final in java? - 스택오버플로우](https://stackoverflow.com/questions/34087724/what-is-the-difference-between-immutable-and-final-in-java)
