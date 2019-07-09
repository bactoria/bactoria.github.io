---
layout: post
title:  "Vue + Firestore 무한스크롤"
categories: Vue
author: bactoria
---

Vue 스터디에서 게시판을 만들기로 함. 게시판은 pagination이 아닌, 무한 스크롤 방식으로 하기로 했다.

&nbsp;

### 무한스크롤 라이브러리 선택

[asesome-vue](https://github.com/vuejs/awesome-vue)에서 보면 무한 스크롤이 있겠다 싶어 봤더니 역시나 있었다.

![image](https://user-images.githubusercontent.com/25674959/57755870-ed98ef00-772c-11e9-986c-d4557f57125c.png)

이 중에 Star 수가 가장 많은 것은 `vue-infinite-scroll`이지만, 나는 `vue-infinite-loading`을 선택했다. 그 이유는 Star 수도 적지 않았고, 최근 커밋이 2개월 전이며(vue-infinite-scroll는 무려 2년 전이다), 데모를 볼 수 있어서 선택했다.

&nbsp;

### How to Use

[https://peachscript.github.io/vue-infinite-loading/guide/#installation](https://peachscript.github.io/vue-infinite-loading/guide/#installation) 에 가이드가 잘 나와있다. 따라하면 된다.

&nbsp;

### with FireStore

API에게 해당 page의 게시글을 응답받기 위해서는 일반적으로 page 번호를 포함하여 요청한다.

FireStore 약간 다른데, page 번호를 보내지 않고 마지막으로 받은 doc를 포함하여 요청한다.

&nbsp;

[쿼리 커서로 데이터 페이지화 - firestore](https://firebase.google.com/docs/firestore/query-data/query-cursors)에 아래와 같이 나와있다.
```javascript
// 쿼리 커서를 limit() 메소드와 결합하여 쿼리를 페이지화합니다. 
// 예를 들어 일정 분량의 마지막 문서를 다음 분량의 커서 시작점으로 사용합니다.

var first = db.collection("cities")
        .orderBy("population")
        .limit(25);

return first.get().then(function (documentSnapshots) {
  // Get the last visible document
  var lastVisible = documentSnapshots.docs[documentSnapshots.docs.length-1];
  console.log("last", lastVisible);

  // Construct a new query starting at this document,
  // get the next 25 cities.
  var next = db.collection("cities")
          .orderBy("population")
          .startAfter(lastVisible)
          .limit(25);
});
```

`lastVisible` : 마지막으로 받아온 문서이다.
`next` : 다음 page를 받을 때 `lastVisible`를 이용한다.

&nbsp;

위 코드를 어떻게 써야할지 감을 못 잡아서 아래처럼 짰다.

**api.js**
```javascript
//...
const boardLimit = 10;

export const board = {
    fetchs(page) {
        if (page === 'first') {
            return firebase.firestore().collection('boards')
                .orderBy("date", "desc")
                .limit(boardLimit)
                .get()
                .then(docs => {
                    docs.page = docs.docs[docs.docs.length - 1]
                    return docs
                })
        }

        return firebase.firestore().collection('boards')
            .orderBy("date", "desc")
            .startAfter(page)
            .limit(boardLimit)
            .get()
            .then(docs => {
                docs.page = docs.docs[docs.docs.length - 1]
                return docs
            })
    }

    //...
}
```

&nbsp;

무한스크롤을 내리다가 게시판을 클릭하고 뒤로가기를 했을 경우, 앞에서 스크롤 했던 게시글들을 그대로 유지하기로 정했다. 이를 위해서는 Vuex에 게시글들을 저장해야 한다.이 때, 주의해야 할 것은 page 또한 Vuex에서 관리를 해야한다는 것이다. 그렇지 않으면 이전에 불러왔던 게시글 밑으로 게시글이 처음부터 추가되는 기이한 현상이 발생한다.

이런 경험을 하기 전에 논리적으로 생각할 수 있었으면 좋았을텐데.. 논리적 사고가 부족함을 다시 한번 느꼈다.

&nbsp;

**store/state.js**
```javascript
const state = {
    //...
    page: 'first'
}

export default state
```
`page: Object` 로 하지 않은데는 이유가 있다.

마지막 게시글까지 스크롤을 내려을 때 page에 null이 담기는데, 이 경우에는 처음 스크롤과, 마지막 스크롤의 경우를 분기하기가 까다로울 것 같다. (분기하지 않으면 말그대로 무한으로 스크롤이 가능하다..)

더 좋은 방법이 있을 것 같긴 하지만, 이 부분은 스터디 모임 때 생각해보기로