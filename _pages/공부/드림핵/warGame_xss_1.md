---
title : "드림핵 워게임 xss-1"
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
![홈]({{ '/assets/img/xss1_0.png' | relative_url }})

**vuln 페이지**
![vuln]({{ '/assets/img/xss1_1.png' | relative_url }})

**memo 페이지**
![memo]({{ '/assets/img/xss1_2.png' | relative_url }})

**flag 페이지**

![flag]({{ '/assets/img/xss1_3.png' | relative_url }})

# 문제 풀이
-------------

memo페이지와 flag 페이지를 이용해서 쿠키를 얻어낼것이다.

![xss4]({{ '/assets/img/xss1_4.png' | relative_url }})

flag페이지에 들어가서 아래 익스플로잇 코드를 입력한다.
```
<script>location.href = "/memo?memo=" + document.cookie;</script>
```
그러면 alert 창에서 good이 뜬다.

바로 memo 페이지를 들어가면 , 사용자의 쿠키 정보값을 알 수 있다.

![xss5]({{ '/assets/img/xss1_5.png' | relative_url }})

flag 값이 출력되었다!