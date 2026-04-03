TeloPon을 더욱 편리하게 만드는 「공식 확장 플러그인（연동 기능）」의 최신 버전입니다!
필요한 기능의 파일（`.py`）만 다운로드하여 TeloPon에 추가하세요.
**※ V2.14b 버전부터 사용할 수 있습니다.**

## 📦 수록 플러그인

### 🆕 🔊 VOICEVOX 읽기 (`voicevox_plugin.py`)
AI가 출력한 텔롭의 텍스트를 **VOICEVOX** 음성 합성 엔진으로 실시간 읽어줍니다.
* **다양한 캐릭터 보이스**: VOICEVOX에 등록된 전체 캐릭터 + 스타일（성격）에 대응. 드롭다운으로 간편 선택.
* **아이콘 표시**: 선택한 캐릭터의 아이콘이 설정 화면에 표시됩니다.
* **세밀한 음성 조정**: 속도·음량·음높이·억양·발화 전 무음의 5가지 파라미터를 실시간 조정.
* **TOPIC 읽기**: ON이면 텔롭 상단 텍스트（TOPIC）도 함께 읽어줍니다.
* **텔롭 표시 딜레이**: 읽기와 OBS 텔롭 표시 타이밍을 맞출 수 있습니다.
* **자동 연결 확인**: 실행 시 VOICEVOX 연결을 확인. 연결 불가 시 자동으로 OFF.

👉 [상세 사용 가이드](../plugins/voicevox_plugin.md)

> 💡 **필요 환경**: [VOICEVOX](https://voicevox.hiroshiba.jp/) 를 설치하고 TeloPon과 동시에 실행하세요.

### 💬 Discord 실시간 연동 (`discord_integration.py`)
Discord 서버의 지정 채널 코멘트를 실시간으로 가져와 AI에 전달합니다.
* **초간단 설정**: Bot 토큰을 입력하고 버튼을 누르기만 하면 초대 URL이 자동 생성됩니다.
* **완전 실시간**: 타임래그 없이 즉시 코멘트를 수집합니다.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

### 🏢 Slack 코멘트 연동 (`slack_integration.py`)
Slack 워크스페이스의 지정 채널 코멘트를 실시간으로 가져와 AI에 전달합니다.
* **Socket Mode 대응**: 기업에서도 사용하는 최신 통신 방식으로 지연 제로의 완전 실시간 수신을 실현.
* **이름 자동 변환**: Slack 고유의 사용자 ID를 자동으로 「발언자 표시명」으로 변환하여 AI에 전달합니다.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

---

## 🛠️ 플러그인 도입 방법（사용법）

1. 이 페이지 맨 아래 **Assets** 에서 원하는 기능의 파일（`.py`）을 클릭하여 다운로드합니다.
2. PC에 있는 `TeloPon-XXX` 폴더를 엽니다.
3. 그 안의 **`plugins`** 폴더에 다운로드한 `.py` 파일을 넣습니다.
4. TeloPon을 실행하면 오른쪽 「🔌 확장 기능」목록에 새로운 연동 기능이 추가됩니다!

※ 설정 방법의 자세한 내용은 각 설정 화면（⚙️）이나 공식 매뉴얼을 참조하세요.

---

## 📋 릴리스 제목（GitHub Releases 용）

```
🔌 TeloPon 공식 확장 플러그인 팩 v1.3 (VOICEVOX·Discord·Slack)
```

