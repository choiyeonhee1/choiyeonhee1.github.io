---
title : "드림핵 워게임 cookie"
tags : 
date : "2026-01-23"
bookmark :  true
---

# 문제 설명
-------------

쿠키로 인증 상태를 관리하는 간단한 로그인 서비스입니다.
admin 계정으로 로그인에 성공하면 플래그를 획득할 수 있습니다.

플래그 형식은 DH{...} 입니다.

**홈 화면**
![홈]({{ 'assets\img\cookie.png' | relative_url }})

# 문제 풀이
-------------

개발자모드를 켜서 username의 vaule값을 admin이라고 입력하고 새로고침하면 ,

![cookie2]({{ 'assets\img\cookie_2.png' | relative_url }})

바로 admin으로 로그인되며 플래그값을 얻을 수 있다.

![cookie3]({{ 'assets\img\cookie_3.png' | relative_url }})