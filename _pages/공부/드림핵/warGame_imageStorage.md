---
title : "드림핵 워게임 image-storage "
tags : 
date : "2026-03-20"
bookmark :  true
---

# 문제 설명
--------------
php로 작성된 파일 저장 서비스입니다.

파일 업로드 취약점을 이용해 플래그를 획득하세요. 플래그는 /flag.txt에 있습니다.

**홈 화면**
![설명]({{ '/assets/img/storage_0.png' | relative_url }})


**list page**
![설명]({{ '/assets/img/storage_1.png' | relative_url }})

**upload page**
![설명]({{ '/assets/img/storage_2.png' | relative_url }})



# 문제 풀이
--------------

우선 메모장에서 간단한 웹 셸을 만들어 cmd.php로 저장한다.

![설명]({{ '/assets/img/storage_3.png' | relative_url }})

업로드 페이지에 들어가서 php 파일을 업로드 하려니까,


![설명]({{ '/assets/img/storage_4.png' | relative_url }})

라는 경고창이 뜨면서 업로드가 안된다. 그리고 내 pc에서 트로이목마를 발견했다고하면서 계속 파일을 삭제시킨다.. 윈도우 보안에서 설정헤도 파일이 계속 삭제된다..

알고보니 내가 작성한 코드는 가장 기본적이고 유명한 웹 셸 패턴이라서 차단당하는거였다.

웹 셸 코드를 새로 작성하고 업로드한다.

![설명]({{ '/assets/img/storage_5.png' | relative_url }})

{% raw %}
왜 새로 작성한 코드는 안걸렸을까?
악성 코드인 system() 함수 주변을 수많은 정상적인 HTML 태그(&lt;form&gt;, &lt;input&gt;,  &lt;body&gt; 등)로 감싸고 있어서 백신 엔진이 파일을 검사할 때, 이 파일의 전체적인 구조가 정상적인 웹 페이지에 가깝다고 판단해서 악성 패턴을 인식하지 못한거라고 한다.

{% endraw %}

그러면 List페이지에 내가 업로드한 파일이 뜬다.

![설명]({{ '/assets/img/storage_6.png' | relative_url }})

cmd.php 를 들어가면 입력하는 칸이 나온다. 문제의 목표는 /flag.txt의 내용을 읽어오는거니까 입력칸에 cat /flag.txt 를 입력하면,

![설명]({{ '/assets/img/storage_7.png' | relative_url }})

플래그 값을 얻을 수 있다.


![설명]({{ '/assets/img/storage_8.png' | relative_url }})