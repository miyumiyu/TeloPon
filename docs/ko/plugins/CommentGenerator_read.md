# 💬 댓글 파일 읽기 (CommentGenerator_read.py)

이 플러그인은 **로컬 PC에 출력된 댓글 데이터(`comment.xml`)를 모니터링하여 시청자 댓글을 실시간으로 AI에게 전달**합니다.

이 기능의 가장 큰 강점은 YouTube뿐만 아니라 **Twitch, 니코니코 생방송, TwitCasting, Mirrativ 등 어떤 스트리밍 플랫폼에서도 작동**하여 AI가 어디서나 댓글을 받아들일 수 있다는 점입니다!

---

## 🛠️ 설정: comment.xml을 출력하는 시스템 만들기

TeloPon 단독으로는 YouTube 이외의 플랫폼에서 댓글을 직접 가져올 수 없습니다. 두 가지 무료 외부 도구를 조합하여 PC에 `comment.xml` 파일을 출력해야 합니다.

### Step 1: 두 개의 필수 애플리케이션 다운로드

1. **Multi Comment Viewer**
   여러 스트리밍 플랫폼의 댓글을 한곳에 모아주는 소프트웨어.
   👉 [Multi Comment Viewer 공식 사이트](https://ryu-s.github.io/app/multicommentviewer)에서 "안정 버전"을 다운로드하여 압축을 해제합니다.
2. **HTML5 Comment Generator**
   수신한 댓글을 파일(XML)로 출력하는 도구.
   👉 [KILINBOX (공식 사이트)](https://lib.kilinbox.net/comegene/index.cgi)에서 최신 버전(예: `hcg_0_0_8b.zip`)을 다운로드하여 압축을 해제합니다.

### Step 2: 두 애플리케이션 연동

1. 압축 해제한 "Multi Comment Viewer" 폴더 안의 `MultiCommentViewer.exe`를 더블클릭하여 실행합니다.
2. 상단 메뉴에서 **"플러그인"**, **"CommentGenerator 연동"**을 클릭합니다.
3. 열리는 설정 창에서 **"CommentGenerator 연동을 활성화"**를 체크합니다.
4. "설정 파일 위치" 옆의 **"찾아보기"** 버튼을 클릭하고 Step 1에서 압축 해제한 "HTML5 Comment Generator" 폴더 안의 **`setting.xml`** 파일을 선택합니다.

### Step 3: comment.xml 출력 확인

1. Multi Comment Viewer에서 "연결 추가"로 스트림 URL을 추가하고 댓글 가져오기를 시작합니다.
2. 댓글이 수신되면 지정한 HTML5 Comment Generator 폴더 안에 **`comment.xml`** 파일이 자동으로 생성(또는 업데이트)됩니다.

설정 완료! 이제 TeloPon이 이 `comment.xml`을 모니터링하도록 합니다.

---

## ⚙️ TeloPon 설정 및 사용법

### 1. 설정 패널 열기
TeloPon 메인 화면 우측의 "확장(플러그인)" 패널에서 **"댓글 파일 읽기"**의 **"⚙️ 설정"** 버튼을 클릭합니다.

![댓글 파일 읽기 설정](../../../images/CommentGenerator_read.png)

### 2. 파일 경로 지정
**"모니터링할 댓글 파일 (XML/TXT)"** 옆의 **"찾아보기..."** 버튼을 클릭합니다.
Step 3에서 확인한 HTML5 Comment Generator 폴더 안의 **`comment.xml`** 파일을 선택합니다.

### 3. 전송 간격(쿨다운) 설정
**"AI에 전송하는 간격(초)"**을 설정합니다 (기본값: `5.0`초 권장).
* 댓글이 올 때마다 AI에게 전송하면 과부하가 걸려 오작동할 수 있으므로, 플러그인은 설정한 초 동안 댓글을 "모아두었다가" 한꺼번에 전송합니다.
* 댓글이 많이 오는 방송에서는 `10`처럼 약간 긴 값을 설정하는 것을 권장합니다.

### 4. 활성화 및 저장
* 화면 상단의 **"댓글 연동 활성화"** 체크박스를 체크합니다.
* **"저장"**을 눌러 창을 닫습니다.
* 메인 화면의 플러그인 배지가 회색 `OFF`에서 녹색 `ON`으로 변경됩니다.

### 5. 라이브 연결 시작
TeloPon 메인 화면의 **"🔴 라이브 연결 시작"** 버튼을 눌러 AI 세션을 시작합니다.
그 후 시청자가 댓글을 달고 `comment.xml`이 업데이트될 때마다, 설정한 간격으로 댓글이 AI에게 전달되고 AI가 반응합니다!

---

## ⚠️ 주의 사항

* **파일 쓰기 충돌**
  매우 드물게 댓글 생성기가 파일에 쓰는 도중 TeloPon이 파일을 읽으려 할 수 있어 일시적인 읽기 오류가 발생할 수 있습니다. 시스템이 자동으로 재시도하므로 방송에는 영향을 주지 않습니다.
* **YouTube 연동 플러그인과 함께 사용할 때**
  "YouTube 연동 도구"와 이 "댓글 파일 읽기"를 **동시에 활성화하면 같은 댓글이 AI에게 두 번 전송됩니다.** YouTube에서 방송할 때는 둘 중 하나만 활성화해 주세요.

---
[⬅️ 플러그인 목록으로 돌아가기](../../../README_ko.md)
