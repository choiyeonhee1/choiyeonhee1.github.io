---
title : "드림핵 워게임 devtools-sources"
tags : 
date : "2026-01-23"
bookmark :  true
---

# 문제 설명
-------------
개발자 도구의 Sources 탭 기능을 활용해 플래그를 찾아보세요.

플래그 형식은 DH{...} 입니다.


# 문제 풀이
-------------

개발자모드를 실행하여 ctrl+shift+f를 실행하여 모든 파일에 대해서 DH라는 단어가 있는지 검색한다.

![devtools]({{ '/assets/img/devtools.png' | relative_url }})

css파일에 주석 처리가 되어있는것 확인. 바로 플래그값을 얻었다.