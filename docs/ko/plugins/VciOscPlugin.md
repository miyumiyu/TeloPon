# 🎮 VCI OSC 텔롭 (VciOscPlugin.py)

> 📥 **[플러그인 다운로드](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/VciOscPlugin.py)**

이 플러그인은 AI가 출력한 텔롭을 **OSC (OpenSound Control)** 프로토콜을 통해 **VirtualCast VCI**에 실시간 전송합니다.
VirtualCast 가상 공간 내에 텔롭을 표시할 수 있습니다.

---

## 🛠️ 준비

### 필요한 것

1. **VirtualCast** (PC 버전) 설치·실행
2. VirtualCast 설정에서 **OSC 수신 기능** 활성화（포트: 19100）
3. 텔롭 표시용 **VCI**가 VirtualCast에 배치

### VirtualCast OSC 설정

1. VirtualCast 타이틀 화면 → **「VCI」** 설정
2. **「OSC 수신 기능」**을 **「활성」** 또는 **「크리에이터만」**으로 설정
3. 수신 포트가 **19100**（기본값）인지 확인

### VCI 수신 스크립트（Lua）

```lua
vci.osc.RegisterMethod("/telopon/telop/text", function(sender, name, args)
    local text = args[1]
    vci.assets.SetText("TextBoard", text)
end, {ExportOscType.String})
```

---

## ⚙️ TeloPon 설정

### 1. 조작 패널 열기

확장 기능 패널에서 **「VCI OSC 텔롭」** → **「조작 패널」** 클릭.

<img width="400" alt="VCI OSC 텔롭 설정 화면" src="../../images/VciOscPlugin.png" />

### 2. OSC 전송 ON

**「OSC 전송 ON」** 체크 시 텔롭마다 자동 전송.

### 3. 연결 설정

| 설정 | 기본값 | 설명 |
|---|---|---|
| **대상 IP** | `127.0.0.1` | VirtualCast PC의 IP |
| **대상 포트** | `19100` | VirtualCast OSC 수신 포트 |
| **OSC 주소** | `/telopon/telop/text` | VCI의 `RegisterMethod`와 일치 |

### 4. 전송 내용 설정

| 설정 | 기본값 | 설명 |
|---|---|---|
| **📌 TOPIC도 전송** | ON | 「TOPIC \| MAIN」형식으로 전송 |
| **🏷️ 배지도 전송** | OFF | `/address/badge`로 별도 전송 |

---

## 📡 전송되는 OSC 메시지

| OSC 주소 | 타입 | 내용 |
|---|---|---|
| `/telopon/telop/text` | String | 텔롭 본문 |
| `/telopon/telop/text/badge` | String | 배지명（옵션） |
| `/telopon/telop/text/window` | String | 윈도우 종류 |

---
