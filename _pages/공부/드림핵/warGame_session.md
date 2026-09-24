---
title : "드림핵 워게임 session-basic"
tags : 
date : "2026-01-23"
bookmark :  true
---

# 문제 설명
-------------
쿠키와 세션으로 인증 상태를 관리하는 간단한 로그인 서비스입니다.
admin 계정으로 로그인에 성공하면 플래그를 획득할 수 있습니다.

플래그 형식은 DH{...} 입니다.

**홈 화면**
![홈]({{ '/assets/img/session_0.png' | relative_url }})

# 문제 풀이
-------------

admin페이지에 접근하면 admin과 guest의 sessionid를 얻을 수 있다.

![session1]({{ '/assets/img/session_1.png' | relative_url }})

돌아와서 sessionid의 value에 아까 얻은 admin의 sessionid를 넣어준다.

![session2]({{ '/assets/img/session_2.png' | relative_url }})

새로고침하면 admin의 플래그값을 얻을 수 있다.

![session3]({{ '/assets/img/session_3.png' | relative_url }})