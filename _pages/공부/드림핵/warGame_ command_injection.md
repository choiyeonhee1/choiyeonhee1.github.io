---
title : "드림핵 워게임 command-injection-1 "
tags : 
date : "2026-03-20"
bookmark :  true
---

# 문제 설명
-----------
특정 Host에 ping 패킷을 보내는 서비스입니다.
Command Injection을 통해 플래그를 획득하세요. 플래그는 flag.py에 있습니다.

**홈 화면**

![설명]({{ 'assets\img\command1_0.png' | relative_url }})

**ping page**

![설명]({{ 'assets\img\command1_1.png' | relative_url }})



# 문제 풀이
------------

사이트에 들어가 input값에 8.8.8.8; ls #을 입력하면 형식과 일치시키라는 박스가 뜬다.

![설명]({{ 'assets\img\command1_2.png' | relative_url }})

개발자모드를 들어가 코드를 확인해보면,

![설명]({{ 'assets\img\command1_3.png' | relative_url }})

input값에 들어가는 입력값 기준을 설정해놓은게 보인다.

그러면 패턴 부분을 지우면 된다.

![설명]({{ 'assets\img\command1_4.png' | relative_url }})

다시 input 값에 8.8.8.8; ls #을 입력하면 이렇게 뜬다.

![설명]({{ 'assets\img\command1_5.png' | relative_url }})

문제의 목표는 flag.py안에 있는 파일을 읽어오는거니까, input값에 8.8.8.8; cat flag.py # 을 입력해준다. (#을 붙이는 이유는 "를 주석처리하기위해서)

그러면 8.8.8.8에 대한 결과와 flag.py 파일을 읽은 결과값이 나온다!

![설명]({{ 'assets\img\command1_6.png' | relative_url }})
