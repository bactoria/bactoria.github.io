---
layout: post
title:  "Comparable vs Comparator"
categories: Java
author: bactoria
---

## Comparable vs Comparator

**Comparable**: 현재 객체에 대해 정렬 가능 (기본정렬)  
**Comparator**: String.class 와 같이, 직접적으로 수정할 수 없는 클래스의 경우에 사용, 동적으로 정렬 기준을 바꿔야 할 경우에 사용

&nbsp;

## Comparable

객체끼리 비교하고 싶은 경우, 해당 클래스에서 Comparable 인터페이스 구현하면 됨.

객체를 만들 때 인터페이스를 구현해서 씀

```java
class Node implements Comparable<Node>{
    String name;
    int age;

    @Override
    public int compareTo(Node that) {
        return Integer.compare(this.age, that.age); // 난 이게 좋아
    }
}
```

- `return this.age == that.age ? 0 : this.age > that.age ? 1 : -1`
    - 코드 더티. 한눈에 이해 안됨.
- `return this.age - that.age`
    - age가 음수이면 스택오버플로우 터짐 ( Integer.MAX_VALUE - (-1) )
- `return Integer.valueOf(this.age).compareTo(that.age)`
    - Good
- `return Integer.compare(this.age, that.age)`
    - 난 이게 더 보기 좋음

&nbsp;

### 한계

- 어떤 Node.class에 대해서 한가지의 방법으로 밖에 정렬 할 수 없음. 
- 여러가지 정렬 기준을 만들 수 없음. (Comparator 써야 함.)

&nbsp;
&nbsp;

## Comparator

PriorityQueue, TreeSet 등의 객체끼리 비교를 관리해야 하는 경우, 생성할 때 기준을 지정해줄 수가 있음.

```java
class AgeComparator implements Comparator<Node> {
	@Override
	public int compare(Node o1, Node o2) {
		return Integer.compare(o1.age, o2.age);
	}
}

public class Main {
    public static void main() {
        Queue<Node> queue = new PriorityQueue<>(new AgeComparator());
    }
}
```

&nbsp;

### Comparator :: 클래스 -> 람다로 변환

```java
class AgeComparator implements Comparator<Node> {
	@Override
	public int compare(Node o1, Node o2) {
		return Integer.compare(o1.age, o2.age);
	}
}

class NameComparator implements Comparator<Node> {
	@Override
	public int compare(Node o1, Node o2) {
		return o1.name.compareTo(o2.name);
	}
}

class Node {
    String name;
    int age;
}

public class Main {
    public static void main() {
        
        //...

        // 1. NameComparator (Basic)
        list.sort(new NameComparator());
        
        // 2. 익명 클래스
        list.sort(new Comparator<Node>() { 
            public int compare(Node o1, Node o2) {
                return o1.name.compareTo(o2.name);
            }
        });

        // 3. 람다식
        list.sort((Node o1, Node o2) -> {
                return o1.name.compareTo(o2.name);
        });

        // 4. 람다식이 한줄이라면 더 생략 가능
        list.sort((o1,o2) -> o1.name.compareTo(o2.name)); 
    }
}
```
