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

## 🏗️ 2. [중요] HTML 구조와 두 개의 텔롭 컨테이너

TeloPon의 표시 화면(`base.html`)에는 **"일반 컨테이너"**와 **"특수 컨테이너"** 두 개의 텔롭 컨테이너가 내장되어 있습니다.
어느 컨테이너를 사용할지는 AI가 지정하는 "윈도우 ID"(아래 참조)에 따라 자동으로 결정됩니다.

### ① 일반 텔롭 컨테이너 (normal group)
일반적으로 화면 우상단에 순서대로 표시되는 표준 텔롭 컨테이너입니다.
```html
<div id="teloop-container" class="윈도우 ID와 레이아웃 ID가 여기에 들어갑니다">
    <div id="teloop-badge"></div>
    <div id="text-wrapper">
        <div class="topic-line">토픽</div>
        <div class="main-line">본문 텍스트</div>
    </div>
</div>
```

### ② 특수 컨테이너 (special group)
화면 하단의 설명 텔롭이나 이름 소개 등, 일반 텔롭과 "독립적으로(겹쳐서)" 표시하고 싶을 때 사용합니다.
```html
<div id="explain-container" class="윈도우 ID가 여기에 들어갑니다">
    <div id="exp-badge"></div>
    <div id="exp-text-wrapper">
        <div class="topic-line">토픽</div>
        <div class="main-line">본문 텍스트</div>
    </div>
</div>
```

---

## 🪪 3. 사용 가능한 윈도우 ID와 시스템 제어

TeloPon은 미리 정의된 "윈도우 ID"를 사용하도록 AI에게 지시합니다. CSS에서는 각 윈도우 ID에 해당하는 클래스(`.window-xxx`)를 대상으로 디자인 규칙과 시스템 명령을 작성합니다.

### 📌 일반 컨테이너 ID (normal group)
`#teloop-container`에 적용되며, UI의 "일반 표시 시간(초)"에 설정된 시간 후 사라집니다.

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

---

## 🔊 4. 사운드 재생 및 시스템 제어 (CSS 변수)

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

## 📐 5. 레이아웃 ID (텍스트 균형 조정)

윈도우 ID 외에, 일반 컨테이너(`#teloop-container`)는 "레이아웃 ID"도 받습니다. 이것을 사용하여 **토픽과 본문 텍스트 간의 크기 균형**을 조정합니다.

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

## 🎬 6. 애니메이션 구현 (`.visible`)

텔롭이 화면에 나타날 때, 시스템은 컨테이너에 **`.visible` 클래스**를 추가합니다.
이것을 CSS `transition`과 `transform`과 함께 사용하여 부드러운 등장 애니메이션을 만들 수 있습니다.

```css
/* =========================================
   일반 텔롭 (우상단에서 확대되며 페이드인)
   ========================================= */
#teloop-container {
    position: absolute; right: 30px; top: 60px;
    opacity: 0;
    transform: scale(0.8);
    transition: all 0.5s;
}
#teloop-container.visible {
    opacity: 1;
    transform: scale(1);
}

/* =========================================
   설명 텔롭 (하단에서 위로 슬라이드)
   ========================================= */
.window-explain {
    position: absolute; left: 40px; bottom: 40px;
    opacity: 0;
    transform: translateY(30px);
    transition: all 0.6s cubic-bezier(0.2, 0.8, 0.2, 1);
    --window-group: special;
}
.window-explain.visible {
    opacity: 1;
    transform: translateY(0);
}
```

---

## 💡 7. 개발 팁 (디버깅)

AI가 말할 때마다 테마 CSS를 조정하는 것은 번거롭습니다.
디자인을 확인하고 조정하고 싶을 때는 디버그 모드(`-d`)로 TeloPon을 실행하세요.

```bash
python telopon_live.py -d
```

UI 하단에 **"수동 제어 패널"**이 나타납니다.
토픽, 본문 텍스트, 테스트하려는 윈도우 ID를 입력한 후 "강제 표시" 버튼을 누르면 — AI 없이도 원하는 만큼 렌더링 테스트(사운드 재생 확인 포함)를 실행할 수 있습니다.
OBS 브라우저 소스에서 CSS 변경 사항이 실시간으로 적용되는 것을 보면서 디자인을 세밀하게 조정하세요!
