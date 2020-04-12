---
layout: post
title: "로그스태시 mutate filter plugin 파헤치기"
date: 2020-04-12 04:20:12
categories: Elasticsearch
author: bactoria

---

raw data를 Elasticsearch로 밀어넣기 전에, 사용하기 편하도록 가공이 필요하다.  

이는 Logstash의 filter로 해결할 수 있는데, 특히  `mutate filter plugin` 를 사용하면 된다.   

해당 플러그인은 [공식문서](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-convert)에 설명이 잘 되어있다.  

![image](https://user-images.githubusercontent.com/25674959/79037601-a7cefa00-7c0d-11ea-87ef-b3ed43892fc2.png)

( 공식문서에서의 mutation filter plugin 의 `gsub` 옵션에 대한 설명이다. )

공식문서에는 위와 같은 식으로 여러 옵션들이 친절하게 설명되어 있다. 하지만 나는 처음 보았을 때 당황했다.   

아무래도 예시가 없어서 한 눈에 와닿지가 않았던 것 같았는데, 이 참에 공부도 할 겸 하나씩 사용해보며 예시를 남기고자 한다.  

이 글이 mutate를 오랫만에 접할 미래의 나와 Logstash의 mutate를 처음 접하는 사용자에게 조금이나마 도움이 되었으면 한다.  



---

## Mutate filter plugin

한 필드에 대하여 변경을 하는데 옵션은 아래처럼 다양하다.

1. **`convert`**
2. **`copy`** 
3. **`gsub`**  
4. **`join`**
5. **`split`**
6. **`lowercase, uppercase`**
7. **`capitalize`**
8. **`merge`**
9. **`coerce`**
10. **`update`**
11. **`replace`**
12. **`strip`**
13. **`rename`**

&nbsp;

### 1. convert

해당 필드를 형변환한다. 

&nbsp;

**logstash.conf**

```ruby
input {
  stdin{
    codec => json
  }
}

filter {
  mutate {
    convert => {
      "field1" => "integer"
      "field2" => "integer_eu"
      "field3" => "float"
      "field4" => "float_eu"
      "field5" => "string"
      "field6" => "boolean"  
    }
  }
}

output {
  stdout{}
}
```

&nbsp;

#### 1) "field1" => "integer"

<img width="500" alt="스크린샷 2020-04-11 오후 7 47 23" src="https://user-images.githubusercontent.com/25674959/79041765-47e84b80-7c2d-11ea-95a2-158d1d1c42a9.png">

&nbsp;

#### 2) "field2" => "integer_eu"

<img width="500" alt="스크린샷 2020-04-11 오후 7 46 03" src="https://user-images.githubusercontent.com/25674959/79041754-21c2ab80-7c2d-11ea-86cc-f3a3cb2382ca.png">



#### 3) "field3" => "float"

<img width="500" alt="스크린샷 2020-04-11 오후 7 51 01" src="https://user-images.githubusercontent.com/25674959/79041820-c8a74780-7c2d-11ea-8c1c-a0dd846fea2b.png">



#### 4) "field4" => "float_eu"

<img width="500" alt="스크린샷 2020-04-11 오후 7 55 19" src="https://user-images.githubusercontent.com/25674959/79041911-613dc780-7c2e-11ea-8ff9-c59239665f2f.png">



#### 5) "field5" => "string"

<img width="500" alt="스크린샷 2020-04-11 오후 7 58 36" src="https://user-images.githubusercontent.com/25674959/79041975-d7422e80-7c2e-11ea-82ec-d4ae09cf6b99.png">



#### 6) "field6" => "boolean"

<img width="500" alt="스크린샷 2020-04-11 오후 8 01 32" src="https://user-images.githubusercontent.com/25674959/79042042-3f911000-7c2f-11ea-8e54-c2cc59edacec.png">

- false: `0`, `0.0`, `"0"`, `"0.0"`, `"false"`, `"f"`, `"no"`, `"n"`

- true: `1`, `1.0`, `"1"`, `"1.0"`, "true", `"t"`, `"yes"`,` "y"`

&nbsp;

( convert의 경우에는 예시가 공식문서에 아주 잘 나와 있었다. )

&nbsp;

&nbsp;

### 2. copy

필드의 값을 복제한다.  

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    copy => {
      "field1" => "field1_copied"
    }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 08 23" src="https://user-images.githubusercontent.com/25674959/79042145-348aaf80-7c30-11ea-9d58-d180f5605a67.png">

&nbsp;

&nbsp;

### 3. gsub

정규식으로 일치하는 항목을 다른 문자열로 대체한다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    gsub => [
      "field1", "/", "_",
      "field2", "[\\?#-]", "."
    ]
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 16 10" src="https://user-images.githubusercontent.com/25674959/79042284-4c166800-7c31-11ea-9796-dc318dc14bd4.png">

- field1: `/` => `_`    

- field2: `?`, `#`, `-` => `.` 

&nbsp;

&nbsp;

### 4. join

배열을 하나의 문자로 합친다.

&nbsp;ㅂ

**logstash.conf**

```ruby
input { ... }

filter {
   mutate {
     join => { "field1" => "," }
   }
 }

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 24 50" src="https://user-images.githubusercontent.com/25674959/79042395-80d6ef00-7c32-11ea-895a-d52bff6f37b0.png">

&nbsp;

&nbsp;

### 5. split

하나의 문자를 배열로 쪼갠다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    split => { "field1" => "," }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 9 01 31" src="https://user-images.githubusercontent.com/25674959/79043053-a4506880-7c37-11ea-8376-d26faf3f7640.png">

구분자가 해당 필드에 존재하지 않는 경우에도 필드는 배열로 바뀌는 것을 알 수 있다.  

`join` 옵션과 반대 개념이다.

&nbsp;

&nbsp;

### 6. lowercase, uppercase

소문자(대문자) 로 변경

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    lowercase => [ "field1", "field3" ]
    uppercase => [ "field4" ]
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 32 54" src="https://user-images.githubusercontent.com/25674959/79042538-a1537900-7c33-11ea-8b49-e1604efb906f.png">

&nbsp;

&nbsp;

### 7. capitalize

첫 글자를 대문자화, 이외는 소문자화 한다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    capitalize => [ "field1" ]
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 10 08 20" src="https://user-images.githubusercontent.com/25674959/79044598-f8137f80-7c40-11ea-9b14-a7a6caaf06cd.png">

&nbsp;

&nbsp;

### 8. merge

해당 필드 값들을 다른 필드에 포함시킨다.  

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    merge => { "field2" => "field1" }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 37 56" src="https://user-images.githubusercontent.com/25674959/79042631-6aca2e00-7c34-11ea-97ae-7c1ca6ae6aaf.png">

- `"banana"` + `["apple", "melon"]` =>  `["apple", "melon", "banana"]`

- `["banana", "mango"]` + `["apple", "melon"]`  =>  `["apple", "melon", "banana", "mango"]`
- `"banana"` + `"apple"` =>  `["banana", "apple"]`
- `["banana", "mango"]` + "apple" => `["apple", "banana", "mango"]`

&nbsp;

&nbsp;

### 9. coerce

필드 값이 null인 경우에 기본값을 넣어준다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    coerce => { "field2" => "empty" }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 46 54" src="https://user-images.githubusercontent.com/25674959/79042786-97327a00-7c35-11ea-9866-b13f25c83115.png">

단, input에 해당 필드가 key로 존재하지 않는 경우에는 기본값을 넣지 않는다.   

 ex. input이 `{ "username": "bactoria" }` 인 경우 `field1` 필드가 생성되지 않음.

&nbsp;

&nbsp;

### 10. update

해당 필드의 값을 특정값으로 대체한다.  

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    update => { "message" => "%{source_host}: My new message" }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 10 00 40" src="https://user-images.githubusercontent.com/25674959/79044399-ea112f00-7c3f-11ea-93ad-512f9b1f2aa0.png">

input에서 `message` 필드가 들어오지 않는 경우 `message` 필드를 생성하지 않는다.   (`replace`옵션은 필드를 생성함)  

ex) input이 `{ }` , `{ "source_host": "54.17.150.184" }` 인 경우 `field1` 필드가 생성되지 않았다.

&nbsp;

&nbsp;

### 11. replace

해당 필드의 값을 특정값으로 대체한다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    replace => { "message" => "%{source_host}: My new message" }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 56 31" src="https://user-images.githubusercontent.com/25674959/79042958-edec8380-7c36-11ea-995d-6b4f51222bb4.png">

input에서 `message` 필드가 들어오지 않는 경우  `message` 필드를 생성하여 대체한다.  (`update`옵션은 생성하지 않음)  

ex) input이 `{ }` , `{ "source_host": "54.17.150.184" }`인 경우 `message` 필드가 대체되었다.

&nbsp;

&nbsp;

### 12. strip

좌우 공백을 제거한다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
      strip => ["searchKeyword", "username"]
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 9 09 27" src="https://user-images.githubusercontent.com/25674959/79043211-db734980-7c38-11ea-9ef5-d76d3c58c2fd.png">

`trim()` 함수와 같은 기능이다.  

참고로 단어 사이의 공백은 제거되지 않는다. ex) `박토리   아`

&nbsp;

&nbsp;

### 13. rename

해당 필드의 이름을 변경한다.  

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    rename => { "HOSTORIP" => "client_ip" }
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-11 오후 8 51 40" src="https://user-images.githubusercontent.com/25674959/79042873-41aa9d00-7c36-11ea-88f8-e2780828fcf9.png">

&nbsp;

&nbsp;

---



### P.S) mutate 옵션 실행 순서

mutate의 옵션에는 실행순서가 있다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    copy => {
      "field" => "field_copied"
    }
    uppercase => [ "field" ]
  }
}

output { ... }
```



**input / output**

<img width="500" alt="스크린샷 2020-04-12 오전 4 10 59" src="https://user-images.githubusercontent.com/25674959/79052762-adf7c180-7c73-11ea-9bf8-893bfac4d56c.png">

&nbsp;

위의 결과는 순차적으로 처리된다면 { "field": "ABCD", "field_copied": "abcd" }` 일 것이다.

공교롭게도 결과는  `{ "field": "ABCD", "field_copied": "ABCD" }` 이다.   

&nbsp;

그 이유는 옵션 별로 실행 순서가 정해져 있어  `uppercase` 옵션 실행 이후에 `copy` 옵션이 실행되기 때문이다. 

실행 순서에 대해서는 [공식문서](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-proc_order)에 나와있다.

&nbsp;

#### 만약 `copy` 옵션을 먼저 실행하고자 한다면 어떻게 해야 할까?

아래와 같이 mutation을 따로 선언해주면 `copy` 옵션 실행 이후에 `uppercase` 옵션이 실행된다.

&nbsp;

**logstash.conf**

```ruby
input { ... }

filter {
  mutate {
    copy => {
      "field" => "field_copied"
    }
  }
  mutate {
    uppercase => [ "field" ]
  }
}

output { ... }
```

&nbsp;

**input / output**

<img width="500" alt="스크린샷 2020-04-12 오전 4 17 56" src="https://user-images.githubusercontent.com/25674959/79052914-9836cc00-7c74-11ea-99cc-263d66fb5b49.png">

