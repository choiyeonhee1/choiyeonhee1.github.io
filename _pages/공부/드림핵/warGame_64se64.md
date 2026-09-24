---
title : "드림핵 워게임 64se64"
tags : 
date : "2026-01-20"
bookmark :  true
---

# 문제 설명
-----------
"Welcome! 👋"을 출력하는 html 페이지입니다.

소스 코드를 확인하여 문제를 풀고 플래그를 획득하세요.

플래그 형식은 DH{...} 입니다.

**홈화면**

![홈화면]({{ '/assets/img/64se64_0.png' | relative_url }})

# 문제 풀이
--------------
웹페이지에 들어가서 개발자모드 실행하면,

![64se641]({{ '/assets/img/64se64_1.png' | relative_url }})

hidden으로 감춰진 것이 있는데 이 value 값이 Base64로 인코딩 되어있다.
value 값을 디코딩해보면 사진과 같이 파이썬 코드가 나온다.

![64se642]({{ '/assets/img/64se64_2.png' | relative_url }})

파이썬 코드를 vs code에서 돌리면

![64se643]({{ '/assets/img/64se64_3.png' | relative_url }})

결과값이 나온다.

