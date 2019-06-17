---
layout: post
title:  "JPA LinkedHashSet 정렬"
categories: JPA
tags: TODO
author: bactoria
---

* content
{:toc}

## Collections 종류

- List
  - 순서 보장, 중복 가능
- Set
  - 순서 안보장, 중복 불가능

```java

@Entity
public class Comment {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(fetch = FetchType.LAZY, mappedBy = "parent")
    Set<Comment> child = LinkedHashSet<>();

    @ManyToOne(fetch = FetchType.LAZY)
    private Comment parent;

    // ...
```

&nbsp;

**QueryDsl**
```java
    QComment c = QComment.Comment;
    QComment child = new QComment("child");

    query.selectFrom(c)
            .leftJoin(c.child, child).fetchJoin()
            .where(c.post.eq(post))
            .where(c.parent.isNull())
            .orderBy(c.id.asc(), child.id.asc())
            .fetch()
            .stream().distinct().collect(Collectors.toList());
```

댓글에 대한 대댓글들이 순서대로 출력될 줄 알았으나, 순서 보장 안됨.

LinkedHashSet 사용하면 순서 보장될 줄 알았는데 호출할 때마다 순서가 변함. 랜덤으로 긁는 것 같음. 그런데, H2 Console에 찍어보면 순서대로 나오긴 함. 

왜 객체로 담을 때 순서를 보장 안해주는거지? LinkedHashSet이잖아..

&nbsp;

### 해결책

#### 1. List

**Comment.java 일부**
```java
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "parent")
    List<Comment> child = ArrayList<>();
```

리스트로 받으면 마음편함. 순서 보장됨.


#### 2. Set (@OrderBy)

**Comment.java 일부**
```java
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "parent")
    @OrderBy("id asc")
    Set<Comment> child = LinkedHashSet<>();
```

위처럼 @OrderBy 를 추가해주니 순서가 보장 됬음.

&nbsp;

이에 대한 해답은 2가지로 나왔음. 사실 근본적인 문제를 해결하지 못한 느낌임.

LinkedHashSet으로 구현했으면 로우를 넣을 때마다 순서대로 넣어야하는게 아닌가?

확실한건 쿼리자체에서는 정렬되서 나옴. 객체에 넣을때 뭔가 잘못된것 같기도..

TODO :: 나중에 알면 수정
