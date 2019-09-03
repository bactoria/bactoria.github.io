---
layout: post
title: "AWS RDS Connection 안될 때"
categories: AWS
author: bactoria
---

### RDS가 필요하다

### RDS 데이터베이스를 생성했다

![image](https://user-images.githubusercontent.com/25674959/64169912-ca37b280-ce89-11e9-911d-1cc418ac5c10.png)

&nbsp;

### Datagrip으로 접속했다

#### 응 안되

![image](https://user-images.githubusercontent.com/25674959/64169649-39f96d80-ce89-11e9-812b-bf8433e99976.png)

&nbsp;

### 아! 포트 개방 해야지~

![image](https://user-images.githubusercontent.com/25674959/64170067-24d10e80-ce8a-11e9-8647-506cfa3e0aa4.png)

#### 응 그래도 안되


&nbsp;

### 근본적으로 잘못 만들었다.

![image](https://user-images.githubusercontent.com/25674959/64170805-02d88b80-ce8c-11e9-8910-669e23ebb7a2.png)

RDS 데이터베이스를 해당 VPC 외부에서 접속하기 위해서는 위처럼 퍼블릭 액세스를 가능하도록 해주어야 한다.

(Default : 아니요)

&nbsp;

### OK

**`psql -h RDS엔드포인트 bactoria postgres`**

![image](https://user-images.githubusercontent.com/25674959/64215543-4a90fe80-cef0-11e9-9fe6-ceefb0c11a27.png)

&nbsp;

![image](https://user-images.githubusercontent.com/25674959/64171006-7ed2d380-ce8c-11e9-9863-0bd19c096371.png)

&nbsp;

이거 해결하려고 2시간 동안 찾아다녔다. 당연히 보안규칙 문제일 줄 알았는데!!

사실 몇 달 전에도 헤매다가 해결했던 건데... 역시 기록을 안하니 까먹는다
