---
title : "[버그헌팅초급] 웹 기초 "
date : "2026-09-21"
bookmark :  true
---

#  웹 기초  (HTTP / URI / DNS)

## 웹 개념
웹은 HTTP로 정보를 주고받는 서비스임. 정보를 주는 쪽이 서버, 받는 쪽이 클라이언트임.
개인정보(주소, 카드번호 등)가 오가기 때문에 보안이 중요함.

## 프론트/백 구분
- 프론트엔드(Front-end): 눈에 보이는 부분 (HTML/CSS/JS)
- 백엔드(Back-end): 요청을 처리하는 부분
- `404`: 리소스(Resource)를 못 찾았다는 뜻. 리소스는 웹에 있는 자산, 파일 같은 것을 의미함

## HTML / CSS / JS 역할
- HTML: 뼈대
- CSS: 스타일
- JS: 동작
클라이언트 사이드 취약점(XSS 등)을 다룰 때 JS/HTML 이해가 필수임. MDN 문서를 참고하면 도움이 됨.

## URI 구조
`스킴://유저@호스트:포트/패스?쿼리#프래그먼트`

- 스킴(Scheme): http, https, ftp 등 접근 프로토콜
- 유저(User): 거의 생략됨
- 호스트(Host): 도메인/IP
- 포트(Port): 생략 시 http는 80, https는 443
- 패스(Path): 경로
- 쿼리(Query): `?key=value` 형태의 파라미터
- 프래그먼트(Fragment): `#` 뒤에 오는 문서 내 특정 위치 식별자

URI 구조를 개발자가 잘못 이해하고 처리하면 취약점으로 이어질 수 있음.

## HTTP 메시지 구조
- 요청(Request)/응답(Response) 모두 헤더(Header) + 바디(Body)로 구성됨
- 줄바꿈은 CRLF(Carriage Return + Line Feed, `\r\n`)로 구분됨
- 첫 줄은 시작줄(Start Line), 이후는 헤더, 헤더 끝은 빈 CRLF 한 줄로 표시됨
- 바디는 헤더 뒤에 오는 실제 데이터임

툴을 사용하면 이 부분이 자동으로 처리되어 평소엔 신경 쓸 일이 적지만, 프로토콜 관련 취약점을 다룰 때는 알아야 함.

## HTTP vs HTTPS
- HTTP는 평문 통신이라 중간에서 탈취되면 내용이 노출됨
- HTTPS는 HTTP에 TLS 암호화를 적용한 방식이라 탈취되어도 내용을 읽을 수 없음

## DNS
도메인 이름을 IP 주소로 바꿔주는 시스템임.

**www.google.com 접속 흐름**
1. 로컬 DNS 서버(Local DNS Server)에 IP를 물어봄 (캐시가 있으면 바로 반환됨)
2. 없으면 로컬 DNS 서버가 루트 DNS 서버(Root DNS Server)에 물어봄
3. 루트 DNS 서버가 `.com` 담당 서버 위치를 알려줌
4. `.com` 서버가 `google.com` 담당 서버 위치를 알려줌
5. 거기서 `www`의 IP를 받아 최종 접속함

로컬 DNS 서버는 통신사 기본값이지만 사용자가 변경할 수 있음 (예: 구글 DNS `8.8.8.8`)

**A 레코드(A Record) vs CNAME(Canonical Name)**

| 구분 | 연결 방식 | 특징 |
|---|---|---|
| A | 도메인 → IP 직접 연결 | 속도가 빠름, IP가 바뀌면 매번 수정해야 함 |
| CNAME | 도메인 → 다른 도메인(별칭) 연결 | IP가 유동적일 때 유용함, 한 단계를 더 거침 |

DNS 설정이 미흡하면 서브도메인 테이크오버(Subdomain Takeover)라는 취약점으로 이어질 수 있고, 관련 제보도 꽤 많은 편임.
