---
title : "드림핵 워게임 simple-sql"
tags : 
date : "2026-02-12"
bookmark :  true
---

# 문제 설명
-----------
로그인 서비스입니다.
SQL INJECTION 취약점을 통해 플래그를 획득하세요. 플래그는 flag.txt, FLAG 변수에 있습니다.

**홈 화면**

![설명]({{ 'assets\img\simple_sql_0.png' | relative_url }})

**login page 코드**
```
#!/usr/bin/python3
from flask import Flask, request, render_template, g
import sqlite3
import os
import binascii

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

DATABASE = "database.db"
if os.path.exists(DATABASE) == False:
    db = sqlite3.connect(DATABASE)
    db.execute('create table users(userid char(100), userpassword char(100));')
    db.execute(f'insert into users(userid, userpassword) values ("guest", "guest"), ("admin", "{binascii.hexlify(os.urandom(16)).decode("utf8")}");')
    db.commit()
    db.close()

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    db.row_factory = sqlite3.Row
    return db

def query_db(query, one=True):
    cur = get_db().execute(query)
    rv = cur.fetchall()
    cur.close()
    return (rv[0] if rv else None) if one else rv

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        userid = request.form.get('userid')
        userpassword = request.form.get('userpassword')
        res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
        if res:
            userid = res[0]
            if userid == 'admin':
                return f'hello {userid} flag is {FLAG}'
            return f'<script>alert("hello {userid}");history.go(-1);</script>'
        return '<script>alert("wrong");history.go(-1);</script>'

app.run(host='0.0.0.0', port=8000)

```

# 문제 풀이
------------

로그인 페이지 소스코드를 보자.

```
@app.route('/login', methods=['GET', 'POST'])
def login():
if request.method == 'GET':
return render_template('login.html')
else:
userid = request.form.get('userid')
userpassword = request.form.get('userpassword')
res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
if res:
userid = res[0]
if userid == 'admin':
return f'hello {userid} flag is {FLAG}'
return f'<script>alert("hello {userid}");history.go(-1);</script>'
return '<script>alert("wrong");history.go(-1);</script>'

app.run(host='0.0.0.0', port=8000)
```
↓밑에 있는 이 코드가 users테이블에서 이용자가 입력한 userid 값과 userpassword 값이 일치하는 회원정보를 불러오는 코드이다.

```
res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
```

userid는 알고 있으니 비밀번호 검증을 무력화 하는 방법을 써야한다.
아이디 입력창에 admin" -- 을 입력하게되면 비밀번호 검증 부분은 주석처리가 되어서 userid 값만 일치해도 정보를 불러오게 된다.


![설명]({{ 'assets\img\simple_sql_1.png' | relative_url }})

pw칸엔 아무거나 넣고 로그인을 하게되면,

![설명]({{ 'assets\img\simple_sql_2.png' | relative_url }})

flag값이 나온다.
