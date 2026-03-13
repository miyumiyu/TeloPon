TeloPon을 더욱 편리하게 만들어주는 「공식 확장 플러그인（연동 기능）」 첫 번째 팩입니다！
필요한 기능의 파일（`.py`）만 다운로드해서 TeloPon에 추가하세요.
**※ v1.05b 버전부터 사용할 수 있습니다.**

## 📦 수록 플러그인

### 💬 Discord 실시간 연동 (`discord_integration.py`)
Discord 서버의 지정 채널 댓글을 실시간으로 가져와 AI에 전달합니다.
* **초간단 설정**: Bot 토큰을 입력하고 버튼을 누르기만 하면 초대 URL이 자동으로 생성됩니다.
* **완전 실시간**: 타임 래그 없이 즉시 댓글을 수신합니다.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

### 🏢 Slack 댓글 연동 (`slack_integration.py`)
Slack 워크스페이스의 지정 채널 댓글을 실시간으로 가져와 AI에 전달합니다.
* **Socket Mode 지원**: 기업에서도 사용하는 최신 통신 방식으로, 지연 없는 완전 실시간 수신을 실현합니다.
* **자동 이름 변환**: Slack 고유의 사용자 ID를 자동으로 「발언자의 표시 이름」으로 변환하여 AI가 읽어줍니다.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

---

## 🛠️ 플러그인 설치 방법

1. 이 페이지 맨 아래의 **Assets** 에서 원하는 기능의 파일（`.py`）을 클릭하여 다운로드합니다.
2. 컴퓨터에 있는 `TeloPon-XXX` 폴더를 엽니다.
3. 그 안의 **`plugins`** 폴더 안에 다운로드한 `.py` 파일을 넣습니다.
4. TeloPon을 실행하면 오른쪽 「🔌 확장 기능」 목록에 새 연동 기능이 추가되어 있습니다！

※ 설정 방법의 자세한 내용은 각 설정 화면（⚙️）이나 공식 매뉴얼을 참고해 주세요.

---

## 📋 릴리스 제목 (GitHub Releases 용)

```
🔌 TeloPon 공식 확장 플러그인 팩 v1.1 (Discord & Slack)
```
