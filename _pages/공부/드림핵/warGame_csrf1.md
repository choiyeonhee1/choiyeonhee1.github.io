---
title : "드림핵 워게임 csrf-1 "
tags : 
date : "2026-02-06"
bookmark :  true
---

# 문제 설명
-----------
여러 기능과 입력받은 URL을 확인하는 봇이 구현된 서비스입니다.

CSRF 취약점을 이용해 플래그를 획득하세요.

**홈 화면**

![설명]({{ 'assets\img\csrf1_0.png' | relative_url }})

**vuln page**

![설명]({{ 'assets\img\csrf1_1.png' | relative_url }})

**memo page**

![설명]({{ 'assets\img\csrf1_2.png' | relative_url }})

**admin page**

![설명]({{ 'assets\img\csrf1_3.png' | relative_url }})


**flag page**

![설명]({{ 'assets\img\csrf1_4.png' | relative_url }})




# 문제 풀이
------------

로컬호스트에 위치하는 이용자가 /admin/notice_flag 페이지에 방문해야하고 userid가 admin이어야한다.
flag 페이지에서 <img src=>를 이용하여 코드를 작성한다.

```
 <img src= "/admin/notice_flag?userid=admin"/ >
 ```

![설명]({{ 'assets\img\csrf1_5.png' | relative_url }})

제출을 누르고 홈으로 돌아가 memo 페이지를 들어가보면 관리자가 /admin/notice_flag 페이지를 방문한 것을 확인할 수 있다.

![설명]({{ 'assets\img\csrf1_6.png' | relative_url }})

flag값이 나왔다!