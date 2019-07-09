---
layout: post
title:  "ModelAttribute와 RedirectAttributes"
categories: Spring
author: bactoria
---

스프링MVC 서적을 보다가 다음의 코드를 보고 의문이 생겼다.

```java
    @PostMapping("/modify")
    public String modify(BoardVO board, @ModelAttribute Criteria criteria, RedirectAttributes rttr) {
        log.info("modify: {}", board);
        if (boardService.modify(board)) {
            rttr.addFlashAttribute("result", "success");
        }
        rttr.addAttribute("pageNum", criteria.getPageNum());
        rttr.addAttribute("amount", criteria.getAmount());
        return "redirect:/board/list";
    }
```

post method로 게시판을 수정하고, /board/list로 리다이렉트한다. 리다이렉트 할 때는 pageNum, amount가 queryString으로 추가될 것이다.

**Request**
- Request URL: http://localhost:8080/board/modify
- Request Method: POST
- Content-Type: application/x-www-form-urlencoded
- FormData
    + pageNum: 1
    + amount: 10
    + bno: 4497525
    + title: 제목
    + content: 내용
    + writer: 주노

**Response**
- Status Code: 302
- Location: `/board/list?pageNum=1&amount=10`

**Redirect**
- Request URL: `http://localhost:8080/board/list?pageNum=1&amount=10`
- Request Method: GET

&nbsp;

~~그런데 위에서 @ModelAttribute를 사용한 이유는 무엇일까. 어노테이션을 없애도 정상적으로 동작한 것 같은데~~

=> Criteria는 @ModelAttribute 를 생략해도 Model에 등록된다. 파라미터가 Object일 때 어노테이션 없어도 Model에 등록됨. (그래도 명시해주는 것이 좋음) 또한, @ModelAttribute에 대해서 제대로 이해를 못했던 것 같다. 모델에 오브젝트를 추가하기 이전에, 일단 QueryString을 객체로 바인딩 해준다는 것을 잠시 망각했다. 

String, Long은 @ModelAttribute로 받지 못함. Java Beans 규칙(기본생성자, getter, setter 존재)에 맞는 객체만 받을 수 있기 때문. (애초에 @RequestParam 으로 받으면 됨.)

&nbsp;

## RedirectAttributes

### redirectAttributes.addAttributes();

```java
//...
rttr.addAttributes("pageNum", 5); 
rttr.addAttributes("amount", 10); 
return "redirect /board/list";
```

=> Location : /board/list?pageNum=5&amount=10 

리다이렉트 시, RedirectAttribute들이 QueryString으로 변환되기 때문에 첫번째 파라미터는 String형태만 가능하다.

&nbsp;

### redirectAttributes.addFlashAttributes();

```java
//...
rttr.addFlashAttribute(criteria);
return "redirect:/board/list";
```

=> Location : /board/list

Flash?? => 한번 사용하고 사라진다.

addAttributes와의 차이점

1. QueryString으로 변환되지 않는다.
2. HttpSession으로 전송된다.
3. 한번 사용하고 사라진다. (새로고침하면 데이터 사라짐)
4. 객체로 넘겨줄 수 있다. (addAttribute는 String만 가능)
5. 리다이렉트된 Method의 Model에 자동으로 등록된다.
은 Object를 HttpSession으로 등록한다. Flash 말 그대로 한번 꺼내면 사라진다. 새로고침하면 데이터 사라짐. 게시글 수정 완료 시 Modal을 여는 조건으로 책에서는 사용됬음.
