---
layout: post
title: "SOLID Principles"
categories: 
tags: SOLID
author: bactoria
---

# SOLID Principles

&nbsp;

## Single Responsibility Principle (단일책임원칙)

클래스를 변경하는 이유는 단 한 가지여야 한다.

일단 클래스가 변경되는 이유를 보면 시스템에 대한 변경 요구가 들어올 때 발생한다.

시스템에 대한 변경 요구를 만족시키기 위해 해당 책임을 가진 클래스의 수정이 일어나는 것이다.

그럼 변경 이유가 2가지일 경우에는 무슨 문제가 생기길래 SRP가 생긴걸까

예시로 A클래스에 변경 이유가 2가지(a,b) 있다고 가정하자.

이 때 제품의 기능에 어떤 변경이 필요할 때, a의 이유로 인해 A클래스가 변경되어야 해서 a를 변경했다고 하자. 이 때 b의 이유로 구현했던 것들이 정상적으로 동작하지 않을 수도 있다.

창문이 내려가지 않아서 정비사에게 창문 수리를 맡겼다. 다음날 차를 다 고쳤다고 문자가 왔다. 정비소에 가서 창문을 내려보니 잘 내려가서 기뻤다. 그래서 시동을 걸었더니 시동이 안걸린다. 변경해달라고 요청하지 않은 부분이 변경된 것이다.

“같은 이유로 변하는 것은 하나로 모으자!, 
	다른 이유로 변하는 것은 분리하자!”	

같은 이유로 변해야 하는 것들에 대해 응집성을 높이고, 다른 이유로 변경하는 것들에 대한 결합성을 낮추세요



또한 변경 이유가 2가지라면 의존해야 하는 객체도 많아질 것이다. 



a의 이유로 인해 A클래스를 변경했는데, 기존에 잘 사용하던 다른 기능에 영향을 줄 수 있다.

(테스트 할 때 private 테스트가 많다. 이 private 메서드를 테스트하고싶다. 어떻게 해야할까?)
=> private 메서드를 테스트해야 하는 메서드라고 생각하면 SRP를 위반하고 있지 않은지를 의심해보아야 한다.

![image](https://user-images.githubusercontent.com/25674959/70489071-f8d6c080-1b3d-11ea-9dc5-f6ea31ffc766.png)


&nbsp;
&nbsp;
&nbsp;

## Open-Close Principle (개방 폐쇄 원칙)

> 확장에는 열려 있어야 하고, 변경에는 닫혀있어야 한다.  

기능을 변경 또는 확장할 수 있으면서 그 기능을 사용하는 코드는 수정하지 않아야 한다.

기능을 사용하는 코드를 수정하지 않으면서 어떻게 기능을 변경/확장이 가능한지 .. 무슨 말을 하는건지 헷갈렸는데, 다형성을 생각하면 된다.

하나의 인터페이스의 구현체가 여러개 있다고 하자. 이 때 새로운 기능 (타입 추가) 이 필요하다면 기존에 만들어주었던 코드를 수정하지 않고 인터페이스의 구현체를 새로 만들면 된다.

기존 코드에 아무런 영향을 미치지 않고 새로운 객체 유형과 행위를 추가할 수 있는 객체지향의 특성이다.

> Employee클래스에 임직원/아르비이트 를 구분하기 위해 boolean 변수인 hourly를 넣어둘 수 있다. hourly 변수를 기반으로 메서드 내에서 타입을 명시적으로 구분하는 방식은 객체지향을 위반하는 것이다.   

> 타입 변수를 이용한 조건문의 분기를 객체지향에서는 다형성으로 개체한다. (Replace Type Code with Class) 클라이언트가 객체의 타입을 확인한 후 적절한 메서드를 호출하는 것이 아니라 객체가 메시지를 처리할 적절한 메서드를 선택한다. - 오브젝트  

> 설계를 할 때 무조건 다형성으로 만드는 것이 옳은 것은 아님.   

> 변경에 있어서 만약 타입이 추가될 여지가 많다면 다형성이 유리하다. 기존의 코드를 건드리지 않고 타입만 추가하면 되기 때문이다. 하지만 이 경우 새로운 오퍼레이션을 추가하기 위해서는 모든 타입들에게 추가시켜주어야 하기 때문에 만약 오퍼레이션을 추가할 확률이 높다면 추상 데이터 타입이 유리하다. - 오브젝트  

폐쇄는 완벽할 수 없다. 일반적으로, 얼마나 닫혀있든지 간에 닫혀있지 않은 것에 대한 변경은 항상 존재하기 때문이다.

따라서 비즈니스에 대한 변경 가능성을 추측하고 그 변경에 대비하기 쉽도록 추상화 하는 것이 바람직하다.

프로그램에서 자주 변경되는 부분을 생각하고 추상화를 잘 하자.

&nbsp;
&nbsp;
&nbsp;

## Liskov Substitution Principle (리스코프 치환 원칙)

> 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.  

중요한 것은 클라이언트 관점에서 행동이 호환되는지 여부다. 행동이 호환될 경우에만 자식 클래스가 부모 클래스 대신 사용될 수 있다 - 오브젝트

> 여기서 요구되는 것은 다음의 치환 속성과 같은 것이다. S형의 각 객체 o1에 대해 T형의 객체 o2가 하나 있고, T에 의해 정의된 모든 프로그램 P에서 T가 S로 치환될 때, P의 동작이 변하지 않으면 S는 T의 서브타입이다 [Liskov88].  

&nbsp;

```java
S s = new o1();  
T t = new o2();
```

**프로그램 P의 일부**
```java
public int calculate(T t) {
	// ...	
}
```

위를 아래와 같이 치환했을 때 프로그램 P가 정상적으로 동작한다면!?

```java
public int calculate(S s) {
	// ...
}
```

S는 T의 서브타입이다.

&nbsp;
&nbsp;

### 리스코프 치환 원칙을 위배하는 예시

> 직사각형(Rectangle)과 정사각형(Square)의 상속 관계는 리스코프 치환 원칙을 위배하는 고전적인 사례 중 하나다

정사각형은 직사각형이다 라는 것을 만족하지만, 리스코프 치환 원칙을 위배할 수 있음.

&nbsp;

**프로그램 P의 일부**

```java
public void resize(Rectangle rectangle, int width, int height) {
	rectangle.setWidth(width);
	rectangle.setHeight(height);
	assert rectangle.getWidth() == width && rectangle.getHeight() == height;
}
```

Rectangle을 상속받은 Square가 있다.  
프로그램 P에서 Square를 Rectangle 대신 사용한다면 프로그램 P가 정상적으로 동작했다고 할 수 있을까 ?

정상적으로 동작한다면 리스코프 치환 원칙을 만족하고, 그렇지 않다면 만족하지 않는다.

&nbsp;

```java
Square square = new Square(10, 10, 10);
resize(square, 50, 100);
```

프로그램 P의 resize 메서드에는 사각형의 너비와 높이가 다르다고 가정하였다. 그래서 square 객체를 넣었을 경우에 assert를 만족시키지 못한다.

resize 메서드는 직사각형은 너비와 높이가 다를 수 있다고 가정하기에, 너비와 높이가 항상 동일하다고 가정하는 정사각형을 넣었을 경우에 문제가 된다.


&nbsp;
&nbsp;
&nbsp;

## Interface Segregation Principle (인터페이스 분리 원칙)

> 인터페이스를 클라이언트의 기대에 따라 분리함으로써 변경에 의해 영향을 제어하는 설계 원칙  

> 이 원칙은 ‘비대한’ 인터페이스의 단점을 해결한다. 비대한 인터페이스를 가지는 클래스는 응집성이 없는 인터페이스를 가지는 클래스다. 즉, 이런 클래스의 인터페이스는 메서드의 그룹으로 분해될 수 있고, 각 그룹은 각기 다른 클라이언트 집합을 지원한다.

인터페이스를 분리하면 변경에 대한 영향을 덜 받을 수 있다.

&nbsp;
&nbsp;
&nbsp;

## Dependency Inversion Principle (의존성 역전 원칙)

```java
@RequiredConsructor
public class Movie {
	  private final AmountDiscountPolicy amountDiscountPolicy;
	  // ...
 
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(amountDiscountPolicy.calculateDiscountAmount(screening));
    }
}
```

상위 수준 클래스인 Movie가 하위 수준 클래스인 AmountDiscountPolicy에 의존한다.  만약 비즈니스가 변경되지 않는다면 문제되지 않겠지만, 변경될 가능성이 있다면 설계는 달라져야 한다.

위의 구조는 결합도가 높아지고 재사용성과 유연성이 낮아진다. Movie를 재사용하기 위해서는 AmountDiscountPolicy 를 함께 재사용 해야 하기 때문이다.

이 설계가 변경에 취약한 이유는 요금을 계산하는 상위 정책이 요금을 계산하는 데 필요한 구체적인 방법에 의존하기 때문이다.

이를 해결하기 위해서는 AmountDiscountPolicy의 추상화인 인터페이스에 의존해야 한다

```java
@RequiredConsructor
public class Movie {
	  private final DiscountPolicy discountPolicy;
	  // ...
 
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

이제 Movie를 재사용 할 때 AmountDiscountPolicy, PercentDiscountPolicy 둘 중 아무거나 사용할 수 있다. 어떤걸 사용할지는 런타임 시점에 결정된다!

유연해졌다. 

&nbsp;

> ‘역전’ 이라는 단어를 사용한 이유는 DIP를 따르는 설계는 의존성의 방향이 전통적인 절차형 프로그래밍과는 반대방향으로 나타나기 때문이다. - 로버트 마틴  

1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 된다. 둘 모두 추상화에 의존해야 한다.
2. 추상화는 구체적인 사항에 의존해서는 안 된다. 구체적인 사항은 추상화에 의존해야 한다.

&nbsp;
&nbsp;
&nbsp;

### 출처

- **SRP**
  - https://code.tutsplus.com/ko/tutorials/solid-part-1-the-single-responsibility-principle--net-36074
  - http://www.gisdeveloper.co.kr/?p=691
  - https://medium.com/@narusas/단일-책임-원칙-the-single-responsibility-principle-번역-c21e0b7307b0

- **OCP**
  - http://wonwoo.ml/index.php/post/1726
  - https://blog.naver.com/PostView.nhn?blogId=jwyoon25&logNo=221615569649&categoryNo=0&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView

- **LSP, ISP, DIP**
  - 오브젝트 - 조영호

