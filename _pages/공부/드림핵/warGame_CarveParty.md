---
title : "드림핵 워게임 Carve Party"
date: "2026-01-23"
bookmark :  true

---

# 문제 설명
----------------
할로윈 파티를 기념하기 위해 호박을 준비했습니다! 호박을 10000번 클릭하고 플래그를 획득하세요!

**홈화면**

![홈화면]({{ '/assets/img/counter_jack_0.png' | relative_url }})


# 문제 풀이 
---------------

개발자 도구를 열어서 HTML을 살펴보자

![counter_jack1]({{ '/assets/img/counter_jack_1.png' | relative_url }})

코드를 보면 jack-target이라는 요소를 클릭할때마다 함수가 실행된다.
클릭할때마다 counter 변수가 1씩 올라간다.
counter가 10000이하이면서 100의 배수일때만 for문이 작동한다.

단순하게 counter 값을 10,000으로 설정한다면?

![counter_jack2]({{ '/assets/img/counter_jack_2.png' | relative_url }})

보라색 글씨만 뜨고 호박 얼굴은 생기지않는다..

그러면 for문을 통해 클릭하는것처럼 만든다면?

![counter_jack3]({{ '/assets/img/counter_jack_3.png' | relative_url }})

![counter_jack4]({{ '/assets/img/counter_jack_4.png' | relative_url }})


10,000번을 클릭한 것으로 나와 호박 얼굴이 제대로 나오게 된다.

정답은 보라색 글씨!