# TeloPon: 테마 & CSS 커스터마이즈 가이드 🎨

TeloPon에서는 "테마"를 사용하여 OBS에 표시되는 텔롭 디자인을 자유롭게 커스터마이즈할 수 있습니다.

TeloPon의 렌더링 시스템은 매우 효율적으로 설계되어 있으며 — **"공통 HTML 뼈대 + 교체 가능한 개별 CSS 디자인"** 아키텍처를 사용합니다. 즉, 새로운 테마를 만들려면 **CSS 파일 하나만 만들면 됩니다**.

---

## 📁 1. 테마 파일 저장 위치

새 테마를 만들려면 `TeloPon/themes/` 폴더 안에 CSS 파일을 만들어 저장합니다.

```text
TeloPon/
 └── themes/
      ├── default.css        ← 표준 테마
      └── my_custom.css      ← 🌟 커스텀 CSS 파일을 여기에 추가하면 됩니다!
```
※ 추가한 CSS 파일은 TeloPon UI의 "⚙️ 텔롭 테마" 드롭다운에 자동으로 표시됩니다.

---

## 🏗️ 2. [중요] HTML 구조와 세 개의 텔롭 컨테이너

TeloPon의 표시 화면(`base.html`)에는 **"일반 컨테이너"**, **"특수 컨테이너"**, **"스테이터스 컨테이너"** 세 개의 텔롭 컨테이너가 내장되어 있습니다.
어느 컨테이너를 사용할지는 AI가 지정하는 "윈도우 ID"(아래 참조)에 따라 자동으로 결정됩니다.

### ① 일반 텔롭 컨테이너 (normal group)
화면 우상단에 여러 텔롭을 "캐스케이드(어긋나게 겹치는)" 형식으로 표시합니다. 텔롭이 나타날 때마다 JavaScript가 동적으로 `.telop-item` 요소를 생성합니다.

```html
<!-- 페이지에 하나 존재하는 "입력 상자" -->
<div id="telop-multi-container">

    <!-- 텔롭이 나타날 때마다 JS가 자동으로 아래와 같은 요소를 생성함 -->
    <div class="telop-item window-variety layout-flat">

        <!-- 텔롭 좌상단의 라벨(배지) -->
        <div class="teloop-badge">AI</div>

        <!-- 텍스트 본체 -->
        <div class="text-wrapper">
            <div class="topic-line">토픽 (상단)</div>
            <div class="main-line">본문 텍스트 (하단)</div>
        </div>

    </div>

</div>
```

> **포인트:** `.telop-item`은 JS가 동적으로 생성합니다. CSS에서 "일반 텔롭 전체"를 대상으로 하려면 `#telop-multi-container .telop-item { ... }` 와 같이 작성합니다.

### ② 특수 컨테이너 (special group)
화면 하단의 설명 텔롭이나 이름 소개 등, 일반 텔롭과 "독립적으로(겹쳐서)" 표시하고 싶을 때 사용합니다. 이 요소는 페이지에 하나만 고정으로 존재합니다.

```html
<!-- 특수 컨테이너 (페이지에 하나 고정) -->
<div id="explain-container">

    <!-- 특수 컨테이너 좌상단 배지 (ID로 지정 가능) -->
    <div id="exp-badge"></div>

    <!-- 텍스트 본체 -->
    <div id="exp-text-wrapper">
        <div class="topic-line" id="exp-topic">토픽 (상단)</div>
        <div class="main-line"  id="exp-main">본문 텍스트 (하단)</div>
    </div>

</div>
```

### ③ 스테이터스 컨테이너 (status group) — 영구 표시
TRPG의 HP/SAN 표시·첫 시청자 카운터·방송 시간 등, **명시적으로 닫을 때까지 화면에 계속 표시되는** 영구 윈도우입니다. 페이지에 하나 고정으로 존재하며, 재전송 시 같은 컨테이너의 내용이 **덮어쓰기 갱신**됩니다.

```html
<!-- 스테이터스 컨테이너 (페이지에 하나 고정) -->
<div id="status-container">

    <!-- 스테이터스 컨테이너 좌상단 배지 (기본은 숨김) -->
    <div id="st-badge"></div>

    <!-- 텍스트 본체 -->
    <div id="st-text-wrapper">
        <div class="topic-line" id="st-topic">제목 (상단)</div>
        <div class="main-line"  id="st-main">본문 (하단)</div>
    </div>

</div>
```

**스테이터스 컨테이너의 특징:**
- `--display-duration: 0` 지정 시 "자동 삭제 없음"이 됩니다
- 방송자는 **OBS의 브라우저 소스에서 직접 조작 가능**: 드래그로 이동 / 휠로 확대축소 / 더블클릭으로 일시 숨김
- 삭제 명령: AI가 `[CMD]SYS:close status`를 발행하거나, 방송자가 "스테이터스 닫아"라고 말함

---

## 🔍 3. CSS에서 사용할 수 있는 셀렉터 완전 레퍼런스

테마 CSS를 작성할 때 "어느 셀렉터를 쓰면 어디에 적용되는지"를 목록으로 정리합니다.

### 일반 텔롭 (normal group) 셀렉터

| 셀렉터 | 종류 | 역할 |
|---|---|---|
| `#telop-multi-container` | ID | 일반 텔롭 전체의 입력 상자. 직접 스타일을 적용하는 경우는 거의 없음 |
| `.telop-item` | 클래스 | 개별 텔롭 요소. 윈도우 ID와 레이아웃 ID도 여기에 추가됨 |
| `.telop-item.visible` | 클래스 | 텔롭이 표시 중인 상태. 애니메이션 제어에 사용 |
| `.teloop-badge` | 클래스 | 텔롭 좌상단의 **빨간 라벨(배지)**. CSS로 여기를 꾸밈 |
| `.teloop-badge.has-text` | 클래스 | 배지에 텍스트가 있는(표시해야 하는) 상태. `display: block`으로 표시 |
| `.no-badge` | 클래스 | 배지가 없을 때 `.telop-item`에 추가됨. 상단 여백 조정에 사용 |
| `.text-wrapper` | 클래스 | 토픽과 본문 텍스트를 묶는 래퍼 |
| `.topic-line` | 클래스 | 토픽 (상단 텍스트) |
| `.main-line` | 클래스 | 본문 텍스트 (하단 텍스트) |
| `.h1` / `.h2` | 클래스 | AI가 강조하고 싶은 단어에 붙이는 강조 클래스. 색상 변경 등에 사용 |

#### 배지 커스터마이즈 예시

```css
/* 빨간 배지를 오리지널 디자인으로 변경하는 예 */
.teloop-badge {
    /* 배지가 숨겨져 있을 때 (기본값) */
    display: none;
    position: absolute;
    top: -45px;
    left: -55px;

    /* 비주얼 디자인 */
    background: #ff0000;   /* 배경색 (빨간색) */
    color: white;
    font-size: 26px;
    font-weight: 900;
    padding: 10px 15px;
    border-radius: 8px;
    border: 3px solid white;
    transform: rotate(-5deg);  /* 약간 기울이기 */
    box-shadow: 4px 4px 10px rgba(0,0,0,0.5);
    white-space: nowrap;
    z-index: 10;
}

/* 배지에 텍스트가 있을 때 (표시) */
.teloop-badge.has-text {
    display: block;
}
```

---

### 특수 컨테이너 (special group) 셀렉터

| 셀렉터 | 종류 | 역할 |
|---|---|---|
| `#explain-container` | ID | 특수 컨테이너 전체의 입력 상자. 윈도우 ID 클래스가 여기에 추가됨 |
| `#explain-container.visible` | ID + 클래스 | 특수 컨테이너가 표시 중인 상태. 애니메이션 제어에 사용 |
| `#exp-badge` | ID | 특수 컨테이너의 **좌상단 배지** (설명 윈도우에서 사용하는 라벨) |
| `#exp-badge.has-text` | ID + 클래스 | 배지에 텍스트가 있는 상태. `display: block`으로 표시 |
| `#exp-text-wrapper` | ID | 특수 컨테이너의 텍스트 래퍼 |
| `#exp-topic` | ID | 특수 컨테이너의 토픽 행 (`.topic-line` 클래스도 함께 붙어 있음) |
| `#exp-main` | ID | 특수 컨테이너의 본문 텍스트 행 (`.main-line` 클래스도 함께 붙어 있음) |

> **ID와 클래스의 차이:** `#`로 시작하는 ID 셀렉터(`#exp-badge` 등)는 페이지에 하나만 있는 요소를 가리킵니다. `.`로 시작하는 클래스 셀렉터(`.teloop-badge` 등)는 여러 요소에 붙을 수 있습니다.

#### 특수 컨테이너 배지 커스터마이즈 예시

```css
/* 설명 윈도우 전용 배지 디자인 예시 */
.window-explain #exp-badge {
    display: none;          /* 기본값은 숨김 */
    position: absolute;
    top: -25px;
    left: 30px;
    background: #00d2ff;    /* 하늘색 */
    color: #000;
    font-weight: 900;
    padding: 5px 25px;
    border-radius: 20px;
    font-size: 24px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.3);
}

.window-explain #exp-badge.has-text {
    display: block;         /* 텍스트가 있으면 표시 */
}
```

---

### 스테이터스 컨테이너 (status group) 셀렉터

| 셀렉터 | 종류 | 역할 |
|---|---|---|
| `#status-container` | ID | 스테이터스 컨테이너 입력 상자. 윈도우 ID 클래스가 여기에 추가됨 |
| `#status-container.visible` | ID + 클래스 | 스테이터스 컨테이너가 표시 중인 상태 |
| `#status-container.dragging` | ID + 클래스 | 방송자가 드래그 중인 상태 (커서 변경 등) |
| `#st-badge` | ID | 스테이터스 컨테이너 좌상단 배지 (기본 `display: none !important;`) |
| `#st-text-wrapper` | ID | 스테이터스 컨테이너 텍스트 래퍼 |
| `#st-topic` | ID | 스테이터스 컨테이너 토픽 라인 (제목) |
| `#st-main` | ID | 스테이터스 컨테이너 메인 텍스트 라인 (본문) |

#### 스테이터스 컨테이너 CSS 예시

```css
.window-status {
    position: fixed; right: 30px; top: 30px;
    min-width: 200px; max-width: 360px;
    padding: 16px 24px;
    background: rgba(20, 30, 45, 0.85); color: #fff;
    border-radius: 12px;
    border-left: 6px solid #00d2ff;
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5);
    transition: opacity 0.4s ease, transform 0.4s ease;
    opacity: 0; transform: translateX(20px);
    z-index: 150;

    --window-group: special;     /* 호환성을 위해 special 지정 (실제 그룹은 "status"로 고정) */
    --display-duration: 0;        /* ★ 0 지정 시 영구 표시 (자동 삭제 없음) */
}
.window-status.visible { opacity: 1; transform: translateX(0); }
.window-status .topic-line {
    font-size: 18px; font-weight: 900;
    color: #00d2ff;
    border-bottom: 2px solid rgba(0, 210, 255, 0.3);
    padding-bottom: 4px; margin-bottom: 8px;
}
.window-status .main-line {
    font-size: 22px; line-height: 1.5;
    white-space: pre-wrap;        /* 줄바꿈(\n)을 그대로 반영 */
    font-weight: 700;
}
```

> **중요**: `window-status`의 그룹은 Python 측에서 `"status"`로 고정되어 있습니다(CSS의 `--window-group` 값과 무관). HP/MP/카운터 등 상시 표시용으로 독립된 컨테이너가 준비되어 있어 일반 컨테이너 및 특수 컨테이너와 독립적으로 표시됩니다.

---

### 이미지 컨테이너 셀렉터

| 셀렉터 | 종류 | 역할 |
|---|---|---|
| `#img-container` | ID | 이미지 표시 입력 상자. 윈도우 ID 클래스가 여기에 추가됨 |
| `#img-container.visible` | ID + 클래스 | 이미지가 표시 중인 상태 |

---

## 🪪 4. 사용 가능한 윈도우 ID와 시스템 제어

TeloPon은 미리 정의된 "윈도우 ID"를 사용하도록 AI에게 지시합니다. CSS에서는 각 윈도우 ID에 해당하는 클래스(`.window-xxx`)를 대상으로 디자인 규칙과 시스템 명령을 작성합니다.

### 📌 일반 컨테이너 ID (normal group)
`.telop-item`에 추가되며, UI의 "일반 표시 시간(초)"에 설정된 시간 후 사라집니다.

* **`.window-simple`**: 가장 표준적이고 심플한 텔롭용.
* **`.window-variety`**: 버라이어티 쇼 스타일의 팝/화려한 디자인용.
* **`.window-news`**: 뉴스 방송 스타일의 격식 있는 디자인용.
* **`.window-cute`**: 손글씨 또는 귀여운/카와이 디자인용.
* **`.window-digital`**: 사이버펑크 또는 디지털 느낌의 디자인용.
* **`.window-reply`**: 시청자 답글 또는 멘션용 (예: "안녕하세요 @사용자님!") — 눈에 띄는 와이드 형식 디자인.

### 📌 특수 컨테이너 ID (special group)
`#explain-container`에 적용됩니다. 시스템에 "이것은 특수 컨테이너"라고 알리려면 CSS에 **`--window-group: special;`을 반드시 지정**해야 합니다.

* **`.window-explain`**: 화면 하단에 크게 표시하는 긴 설명 또는 보충 정보용.
* **`.window-nameplate`**: TV 스타일의 "닉네임 + 이름"을 큰 글씨로 표시하는 용.

### 📌 스테이터스 컨테이너 ID (status group)
`#status-container`에 적용됩니다. **`--display-duration: 0`** 을 지정하면 영구 표시가 됩니다(명시적으로 닫을 때까지 남아 있음).

* **`.window-status`**: HP/SAN/카운터/방송 시간 등 **상시 표시하고 싶은 정보**용.

| 용도 | 예 |
|---|---|
| TRPG 스테이터스 | 탐색자의 이름·직업·HP·SAN 등을 상시 표시 |
| 첫 시청자 카운터 | "첫 시청자 N명" 상시 표시 |
| 방송 시간 | "2시간 15분 경과" 등 |

**조작:**
- 표시 갱신: 같은 `window-status`로 재전송하면 **덮어쓰기 갱신** (같은 컨테이너의 내용이 새로워짐)
- 삭제: AI가 `[CMD]SYS:close status`를 발행 / 방송자가 "스테이터스 닫아"라고 말함
- OBS에서의 조작: 드래그로 이동 / 마우스 휠로 확대축소 / 더블클릭으로 일시 숨김

---

## 🔊 5. 사운드 재생 및 시스템 제어 (CSS 변수)

TeloPon의 가장 큰 특징 중 하나는 **"CSS에서 시스템(Python/JS 측)으로 설정을 역주입"**할 수 있다는 점입니다. 이것은 "CSS 변수(`--`로 시작하는 선언)"를 사용하여 이루어집니다.

### 💡 효과음 재생 (`--sound-file`)
텔롭이 나타날 때 효과음을 재생하려면 CSS에 `--sound-file`을 작성합니다. `TeloPon/sounds/` 폴더의 오디오 파일을 지정합니다.

```css
/* 예시: 버라이어티 윈도우가 나타날 때 "팝" 소리 재생 */
.window-variety {
    background: linear-gradient(135deg, #ff00cc, #333399);
    color: white;
    /* 시스템에 TeloPon/sounds/sound1.mp3 재생을 지시 */
    --sound-file: "sound1.mp3";
}
```

### 💡 컨테이너를 "특수"로 전환 (`--window-group`)
위에서 언급했듯이, `window-explain` 등의 특수 컨테이너 디자인을 만들 때 이 변수를 생략하면 일반 컨테이너로 처리됩니다.

```css
.window-explain {
    position: absolute;
    bottom: 40px;
    /* 이것을 포함하면 "#explain-container"에서 렌더링됨 */
    --window-group: special;
    --sound-file: "sound2.mp3";
}
```

### 💡 표시 시간 강제 재정의 (`--display-duration`)
UI에 설정된 시간(예: 5초)을 무시하고 특정 텔롭을 고정 시간 동안 표시할 수 있습니다.

```css
.window-nameplate {
    /* UI 설정 재정의 — 네임플레이트를 정확히 5초 후 사라지도록 강제 */
    --window-group: special;
    --display-duration: 5;
}
```

---

## 📐 6. 레이아웃 ID (텍스트 균형 조정)

윈도우 ID 외에, 일반 텔롭(`.telop-item`)은 "레이아웃 ID"도 받습니다. 이것을 사용하여 **토픽과 본문 텍스트 간의 크기 균형**을 조정합니다.

* **`.layout-flat`**: 플랫 레이아웃 — 토픽과 본문 텍스트가 동일한 크기.
* **`.layout-big-top`**: 상단 행(토픽)이 더 크고, 하단 행(본문)이 더 작습니다.
* **`.layout-big-bottom`**: 상단 행(토픽)이 더 작고, 하단 행(본문)이 더 크고 눈에 띕니다.

**CSS 사용 예시:**
```css
/* flat이 지정되면 텍스트만 있는 모양으로 테두리 제거 등의 예외를 적용할 수 있음 */
.window-variety.layout-flat .topic-line {
    background: none !important;
    font-size: 36px !important;
}
```

---

## 🎬 7. 애니메이션 구현 (`.visible`)

텔롭이 화면에 나타날 때, 시스템은 컨테이너에 **`.visible` 클래스**를 추가합니다.
이것을 CSS `transition`과 `transform`과 함께 사용하여 부드러운 등장 애니메이션을 만들 수 있습니다.

### 일반 텔롭 애니메이션
일반 텔롭(`.telop-item`)의 기본 애니메이션(확대되며 페이드인)은 `base.html` 내에 고정으로 정의되어 있습니다. 테마 CSS에서 재정의하려면 더 구체적인 셀렉터를 사용합니다.

```css
/* 예: 일반 텔롭을 "오른쪽에서 슬라이드 인"으로 변경 */
#telop-multi-container .telop-item {
    opacity: 0;
    transform: translateX(80px);   /* 오른쪽으로 이동된 상태에서 시작 */
    transition: opacity 0.4s, transform 0.4s;
}
#telop-multi-container .telop-item.visible {
    opacity: 1;
    transform: translateX(0);      /* 제자리로 슬라이드 */
}
```

### 특수 컨테이너 애니메이션
특수 컨테이너(`#explain-container`)의 애니메이션은 테마 CSS에서 완전히 제어합니다. 윈도우 ID 클래스에 `opacity`와 `transform`을 설정하고, `.visible`이 추가될 때 변화시킵니다.

```css
/* =========================================
   설명 텔롭 (하단에서 위로 슬라이드)
   ========================================= */
.window-explain {
    position: absolute; left: 40px; bottom: 40px;
    opacity: 0;                          /* 초기 상태: 투명 */
    transform: translateY(30px);         /* 초기 상태: 약간 아래로 이동 */
    transition: all 0.6s cubic-bezier(0.2, 0.8, 0.2, 1);
    --window-group: special;
}
.window-explain.visible {
    opacity: 1;                          /* 표시: 불투명 */
    transform: translateY(0);            /* 표시: 제자리로 이동 */
}
```

---

## 💡 8. 개발 팁 (디버깅)

AI가 말할 때마다 테마 CSS를 조정하는 것은 번거롭습니다.
디자인을 확인하고 조정하고 싶을 때는 디버그 모드(`-d`)로 TeloPon을 실행하세요.

```bash
python telopon_live.py -d
```

UI 하단에 **"수동 제어 패널"**이 나타납니다.
토픽, 본문 텍스트, 테스트하려는 윈도우 ID를 입력한 후 "강제 표시" 버튼을 누르면 — AI 없이도 원하는 만큼 렌더링 테스트(사운드 재생 확인 포함)를 실행할 수 있습니다.
OBS 브라우저 소스에서 CSS 변경 사항이 실시간으로 적용되는 것을 보면서 디자인을 세밀하게 조정하세요!
