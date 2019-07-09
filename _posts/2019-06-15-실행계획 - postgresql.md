---
layout: post
title:  "실행계획 - postgresql"
categories: SQL
author: bactoria
---

> SELECT 구문 앞에 EXPLAIN 붙이면 실행계획을 볼 수 있음.

&nbsp;
&nbsp;

```sql
EXPLAIN 
SELECT * FROM USER; 
```

&nbsp;

**이용 가능한 게시물의 게시글, 글쓴이, 카테고리를 뽑아오기**

```sql
EXPLAIN
    SELECT  *
    FROM board_data
        INNER JOIN users
            ON board_data.user_id = users.id
        INNER JOIN board
            ON board_data.board_id = board.id
    WHERE board_data.board_data_status = 'AVAILABLE'
    ORDER BY board_data.id DESC;
```

&nbsp;

(실행 계획)
```java
Sort  (cost=27.82..27.82 rows=1 width=4814) // 1)
  Sort Key: board_data.id DESC
  ->  Nested Loop  (cost=0.28..27.81 rows=1 width=4814) // 2)
        ->  Nested Loop  (cost=0.14..19.42 rows=1 width=3766) // 3)
              ->  Seq Scan on board_data  (cost=0.00..10.75 rows=1 width=1145) // 4)
                    Filter: ((board_data_status)::text = 'AVAILABLE'::text)
              ->  Index Scan using users_pkey on users  (cost=0.14..8.15 rows=1 width=2621) // 5)
                    Index Cond: (id = board_data.user_id)
        ->  Index Scan using board_pkey on board  (cost=0.14..8.16 rows=1 width=1040) // 6)
              Index Cond: (id = board_data.board_id)
```

실행 순서 : 4 - 5 - 3 - 6 - 2 - 1 (실행 순서 헷갈리면 [jojoldu님 블로그](https://jojoldu.tistory.com/162?category=761883))

- `Seq Scan on board_data  (cost=0.00..10.75 rows=1 width=1145)` // 4)
  - WHERE 절 수행
  - board_data_status 가 'AVAILABLE' 인 것을 찾는다 ( Seq Scan )
- `Index Scan using users_pkey on users  (cost=0.14..8.15 rows=1 width=2621)` // 5)
  - (Index Scan) 조인 할 대상을 인덱스에 태워서 스캔
- `Nested Loop  (cost=0.14..19.42 rows=1 width=3766)` // 3)
  - 실질적으로 조인 수행
  - 실행속도 :: 선행 테이블(board_data) 사이즈 * 후행 테이블(users) 접근 횟수
- `Index Scan using board_pkey on board  (cost=0.14..8.16 rows=1 width=1040)` // 6)
  - (Index Scan) 조인 할 대상을 인덱스에 태워서 스캔
- `Nested Loop  (cost=0.28..27.81 rows=1 width=4814)` // 2)
  - 실질적으로 조인 수행
  - 실행속도 :: 선행 테이블(board_data) 사이즈 * 후행 테이블(board) 접근 횟수
- `Sort  (cost=27.82..27.82 rows=1 width=4814)` // 1)
  - board_data.id 역순으로 정렬

&nbsp;

