---
title : "드림핵 워게임 file-download-1 "
tags : 
date : "2026-03-20"
bookmark :  true
---

# 문제 설명
--------------
문제 설명
File Download 취약점이 존재하는 웹 서비스입니다.
flag.py를 다운로드 받으면 플래그를 획득할 수 있습니다.

**홈 화면**

![설명]({{ 'assets\img\fileDown1_0.png' | relative_url }})

**upload page**

![설명]({{ 'assets\img\fileDown1_1.png' | relative_url }})

# 문제 풀이
--------------


업로드 페이지에 들어가서 파일 이름에 ../flag.py를 입력하고 파일을 업로드 하면,

![설명]({{ 'assets\img\fileDown1_2.png' | relative_url }})

이렇게 필터링이 되어 업로드가 되지 않는다.

![설명]({{ 'assets\img\fileDown1_3.png' | relative_url }})

정상적으로 파일을 작성하고 업로드를 한다.

![설명]({{ 'assets\img\fileDown1_4.png' | relative_url }})


홈으로 가서 메모 내용을 확인할 수 있다.

![설명]({{ 'assets\img\fileDown1_5.png' | relative_url }})

read페이지는 upload페이지와 다르게 ".."를 필터링하고 있지 않아서, read페이지에서 get파라미터의 filename을 ..로 사용해도 필터링 되지 않는다.

이렇게 name 파라미터값을 ../flag.py 로 바꿔주면,

![설명]({{ 'assets\img\fileDown1_6.png' | relative_url }})

flag.py 의 내용을 읽어올 수 있다.

![설명]({{ 'assets\img\fileDown1_7.png' | relative_url }})

flag값이 출력됐다!