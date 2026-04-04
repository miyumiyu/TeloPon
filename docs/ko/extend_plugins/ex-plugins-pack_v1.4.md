[English](https://github.com/miyumiyu/TeloPon/blob/main/docs/en/extend_plugins/ex-plugins-pack_v1.4.md) | [日本語](https://github.com/miyumiyu/TeloPon/blob/main/docs/ja/extend_plugins/ex-plugins-pack_v1.4.md) | **한국어** | [Русский](https://github.com/miyumiyu/TeloPon/blob/main/docs/ru/extend_plugins/ex-plugins-pack_v1.4.md)

TeloPon을 더욱 편리하게 만드는 「공식 확장 플러그인（연동 기능）」의 최신 버전입니다!
필요한 기능의 파일（`.py`）만 다운로드하여 TeloPon에 추가하세요.
**※ V2.15b 버전부터 사용할 수 있습니다.**

## 📦 수록 플러그인

### 🆕 🎮 VCI OSC 텔롭 (`VciOscPlugin.py`)

<img width="400" alt="VCI OSC 텔롭 설정 화면" src="https://github.com/miyumiyu/TeloPon/blob/main/images/VciOscPlugin.png?raw=true" />

AI가 출력한 텔롭을 **OSC（OpenSound Control）** 프로토콜을 통해 **VirtualCast VCI**에 실시간 전송합니다.
* **VirtualCast 연동**: 가상 공간 내에 텔롭 표시
* **대상 설정**: IP·포트·OSC 주소를 커스터마이즈 가능
* **TOPIC·배지 전송**: 텔롭 본문 외에 TOPIC과 배지도 별도 주소로 전송 가능
* **테스트 전송**: 버튼 하나로 연결 확인

📥 [플러그인 다운로드](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/VciOscPlugin.py)  
👉 [상세 사용 가이드](../plugins/VciOscPlugin.md)

> 💡 **필요 환경**: [VirtualCast](https://virtualcast.jp/)（PC 버전）에서 OSC 수신 기능을 활성화하세요.

### 🔊 VOICEVOX 읽기 (`voicevox_plugin.py`)

<img width="400" alt="VOICEVOX 읽기 설정 화면" src="https://github.com/miyumiyu/TeloPon/blob/main/images/voicevox_plugin.png?raw=true" />

AI가 출력한 텔롭 텍스트를 **VOICEVOX** 음성 합성 엔진으로 실시간 읽어줍니다.
* **다양한 캐릭터 보이스**: 전체 캐릭터 + 스타일에 대응. 드롭다운으로 간편 선택.
* **아이콘 표시**: 선택한 캐릭터 아이콘이 설정 화면에 표시.
* **세밀한 음성 조정**: 5가지 파라미터를 실시간 조정.
* **TOPIC 읽기**: ON이면 TOPIC도 함께 읽기.
* **텔롭 표시 딜레이**: 읽기와 OBS 텔롭 표시 타이밍을 맞출 수 있음.
* **자동 연결 확인**: 연결 불가 시 자동으로 OFF.

📥 [플러그인 다운로드](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/voicevox_plugin.py)  
👉 [상세 사용 가이드](../plugins/voicevox_plugin.md)

> 💡 **필요 환경**: [VOICEVOX](https://voicevox.hiroshiba.jp/) 를 설치하고 TeloPon과 동시에 실행하세요.

### 💬 Discord 실시간 연동 (`discord_integration.py`)
Discord 서버의 지정 채널 코멘트를 실시간으로 가져와 AI에 전달합니다.
* **초간단 설정**: Bot 토큰 입력 → 버튼 클릭으로 초대 URL 자동 생성.
* **완전 실시간**: 타임래그 없이 즉시 수집.
* **아바타 대응**: 코멘트 작성자의 아이콘이 reply 텔롭에 표시.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [플러그인 다운로드](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/discord_integration.py)  
👉 [상세 사용 가이드](../plugins/discord_integration.md)

### 🏢 Slack 코멘트 연동 (`slack_integration.py`)
Slack 워크스페이스의 지정 채널 코멘트를 실시간으로 가져와 AI에 전달합니다.
* **Socket Mode 대응**: 지연 제로의 완전 실시간 수신.
* **이름 자동 변환**: Slack 사용자 ID를 표시명으로 자동 변환.
* **아바타 대응**: 코멘트 작성자의 아이콘이 reply 텔롭에 표시.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [플러그인 다운로드](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/slack_integration.py)  
👉 [상세 사용 가이드](../plugins/slack_integration.md)

---

## 🛠️ 플러그인 도입 방법

1. 이 페이지 맨 아래 **Assets** 에서 원하는 파일（`.py`）을 다운로드.
2. PC의 `TeloPon-XXX` 폴더를 열기.
3. **`plugins`** 폴더에 `.py` 파일을 넣기.
4. TeloPon 실행 → 「🔌 확장 기능」에 추가 완료!

---

## 📋 릴리스 제목（GitHub Releases 용）

```
🔌 TeloPon 공식 확장 플러그인 팩 v1.4 (VCI OSC·VOICEVOX·Discord·Slack)
```
