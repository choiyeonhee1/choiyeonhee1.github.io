---
title : "드림핵 워게임 File Vulnerability Advanced for linux"
tags : 
date : "2026-07-09"
bookmark :  true
---

# 문제 설명
--------------
본 문제는 파일 다운로드 취약점(File Download Vulnerability)이 존재하는 웹 애플리케이션을 대상으로 진행되는 문제이다.

웹 서비스 내의 파일 다운로드 기능을 통해 서버 내부의 민감한 파일이나 프로세스 정보(/proc 등)를 유출할 수 있으며, 이를 통해 관리자 권한 확인 또는 인증에 필요한 API_KEY를 획득하는 것을 목표로 한다.

최종적으로 탈취한 API_KEY를 활용하여 서버 권한을 취득하고, 임의 명령어 실행을 달성하여 플래그(FLAG)를 획득해야 한다.

# 문제 풀이
--------------
main.py의 코드 일부분이다.

![설명]({{ 'assets\img\fileAd_1.png' | relative_url }})

/file 엔드포인트에서는 path 파라미터를 통해 ./files/경로로부터 파일을 읽어오는 역할을 한다. 하지만 path 파라미터에 대한 필터링이 존재하지 않아 Path Traversal 취약점이 발생하고 임의 경로 파일을 다운로드 할 수 있다.

/admin 엔드포인트에서는 데코레이터가 먼저 실행되어 서버에 저장된 API_KEY와 사용자가 보낸 API_KEY 값을 비교하고 일치할 시 cmd 파라미터 값을 가져와서 subprocess.getoutput(cmd) 이 명령어를 셸에서 그대로 실행하고 결과로 받아온다. 그리고 실행 결과를 그대로 응답으로 돌려준다.

```
API_KEY = os.environ.get('API_KEY', None)
```
API_KEY는 환경변수로부터 읽어온다.
파일 다운로드 취약점을 이용하여 /proc/self/environ 파일을 읽어오면 API_KEY를 획득할 수 있다.

/file?path=../../../../../proc/self/environ 을 입력하면

![설명]({{ 'assets\img\fileAd_2.png' | relative_url }})

API_KEY 값이 나온다.

얻은 키 값을 이용하여 /admin 엔드포인트에 접근해 임의 명령어를 실행할 수 있다.
API_KEY 파라미터에 획득한 값을 넣고, cmd 파라미터에 모든 파일 목록을 보여주는 명령어를 입력하면,

![설명]({{ 'assets\img\fileAd_3.png' | relative_url }})

이렇게 파일 목록들이 전부 뜬다.

flag 파일을 보면 x권한만 존재하기 때문에 플래그 파일을 실행해야한다.

cmd 파라미터에 /flag를 입력하면 ,

![설명]({{ 'assets\img\fileAd_4.png' | relative_url }})


flag 값이 나온다!
