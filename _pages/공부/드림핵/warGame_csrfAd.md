---
title : "드림핵 워게임 CSRF Advanced "
tags : 
date : "2026-06-12"
bookmark :  true
---

# 문제 설명
--------------
CSRF 취약점을 통해 관리자 꼐정의 비밀번호를 변경시키고, 로그인을 해보자.

**홈 화면**

![설명]({{ 'assets\img\csrfAd_0.png' | relative_url }})


**change_password page**

 ![설명]({{ 'assets\img\csrfAd_1.png' | relative_url }})

**login page**

![설명]({{ 'assets\img\csrfAd_2.png' | relative_url }})

**vuln page**
이용자가 입력한 값 출력함

![설명]({{ 'assets\img\csrfAd_3.png' | relative_url }})


**flag page**
전달된 URL에 임의 이용자가 접속하게 함.

![설명]({{ 'assets\img\csrfAd_4.png' | relative_url }})


# 문제 풀이
--------------

먼저 vuln 페이지 코드를 보면 ,

![설명]({{ 'assets\img\csrfAd_5.png' | relative_url }})

frame, script, on 키워드를 필터링을 한다. 하지만 &lt; 나 다른 키워드나 태그들은 사용 할 수 있어 csrf 공격이 가능하다.

다음으로 flag 페이지 코드를 살펴보면

![설명]({{ 'assets\img\csrfAd_6.png' | relative_url }})

![설명]({{ 'assets\img\csrfAd_7.png' | relative_url }})

![설명]({{ 'assets\img\csrfAd_8.png' | relative_url }})

```
driver.find_element(by=By.NAME, value= "password" ).send_keys(users[ "admin" ])
```
이 코드는 서버 내부에서 실행되는 코드로, 서버가 자기 메모리에 관라지의 진자 비밀번호를 이미 저장해두고 있다는 뜻이다.
read_url 함수는 셀레늄을 이용해 먼저 서버 코드 내부에 저장되어있던 진짜 관리자 비밀번호(users[ "admin" ])를 꺼내서 자동으로 입력하고 로그인을 한 후 , 내가 입력한 공격주소로 접속하여 관리자의 토큰값이 일치하는지 확인한다. 일치하면 비밇번호 변경 공격이 성공하게 된다.

login 페이지 코드에서,

![설명]({{ 'assets\img\csrfAd_9.png' | relative_url }})

token 구성요소는 이용자 아이디와 IP주소이다.

이를 이용하여 admin token 을 알아낼 수있다.
아이디는 admin이고 IP주소는 로컬호스트 주소이다.

```
from hashlib import md5

username = b"admin"
ip_addr = b"127.0.0.1"
csrf_token = md5(username + ip_addr).hexdigest()
print(csrf_token)
```

파이썬으로 돌려보면

![설명]({{ 'assets\img\csrfAd_10.png' | relative_url }})

이를 가지고 flag 페이지에 들어가서

![설명]({{ 'assets\img\csrfAd_11.png' | relative_url }})

```
<img src="/change_password?pw=admin">
```
를 입력학 제출한 후 ,

login 페이지에 들어가 아이디 admin, 비밀번호 dreamhack을 입력하고 로그인하면

![설명]({{ 'assets\img\csrfAd_12.png' | relative_url }})

화면에 flag값이 출력된다.

![설명]({{ 'assets\img\csrfAd_13.png' | relative_url }})