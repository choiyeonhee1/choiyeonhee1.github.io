---
title : "드림핵 워게임 CSP Bypass Advanced"
tags : 
date : "2026-06-14"
bookmark :  true
---

# 문제 설명
--------------
CSP를 우회하고 XSS 취약점을 통해 임의 이용자의 쿠키를 탈취하자.

앞의 CSP Bypass의 패치된 문제이다.


# 문제 풀이
--------------
먼저 vuln 페이지 코드를 살펴보면 nonce값을 알아야지만 스크립트를 실행할 수 있다.

![설명]({{ '/assets/img/cspAd_1.png' | relative_url }})

그 다음 flag 페이지 코드 중 add_header 함수를 보면 자신의 주소에서만 불러오게했고 자바스크립트는 자신의 주소에서 가져오거나 nonce값이 일치하는 경우에만 실행되도록 규칙을 설정했다. 그 외에 이미지 예외 규칙과 디자인 예외규칙으 설정했다.

![설명]({{ '/assets/img/cspAd_2.png' | relative_url }})

![설명]({{ '/assets/img/cspAd_3.png' | relative_url }})

그런데 add_header에서 CSP 정책 설정 중에 base-uri 정책설정이 빠져있다. &lt;base&gt; 태그를 이용해서 공격을 해보면 될거같다

flag페이지에 공격코드를 넣기 전에, 홈에서 개발자모드를 열어보면

![설명]({{ '/assets/img/cspAd_4.png' | relative_url }})

자바스크립트 파일을 불러오는게 둘다 상대경로로 설정되어있다.
그러면 &lt;base&gt; 태그를 사용해서 내 외부사이트주소로 경로를 지정하면 상대경로들이 전부 내 외부서버주소로 자동 리다이렉트된다.

깃허브에 들어가서 static/js 폴더를 만들고 bootstrap.js 파일을 만들어준다.

![설명]({{ '/assets/img/cspAd_5.png' | relative_url }})

그리고 flag 페이지에 들어가 base 태그를 이용하여
내 외부서버주소를 넣어주고 제출하면, flag값을 볼 수 있다.

![설명]({{ '/assets/img/cspAd_6.png' | relative_url }})

flag 값이 출력되었다.

![설명]({{ '/assets/img/cspAd_7.png' | relative_url }})