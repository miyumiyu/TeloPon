# TeloPon: テーマ・CSS カスタマイズガイド 🎨

TeloPonでは、OBS上に表示されるテロップのデザインを「テーマ（Theme）」として自由にカスタマイズすることができます。

TeloPonの描画システムは非常に効率的で、**「共通のHTML（骨組み）」に対して「個別のCSS（デザイン）」を着せ替える仕組み**になっています。そのため、新しいテーマを作る際は **CSSファイルを1つ作成するだけ** で完了します。

---

## 📁 1. テーマファイルの保存場所

新しいテーマを作るときは、`TeloPon/themes/` フォルダの中に CSSファイルを作成して保存します。

```text
TeloPon/
 └── themes/
      ├── default.css        ← 標準のテーマ
      └── my_custom.css      ← 🌟 ここに自作のCSSファイルを追加するだけ！
```
※追加したCSSファイルは、TeloPon UIの「⚙️ テロップテーマ」のプルダウンから自動的に選べるようになります。

---

## 🏗️ 2. 【重要】HTMLの構造と２つのテロップ枠

TeloPonの画面（`base.html`）には、大きく分けて **「通常枠（normal）」** と **「特別枠（special）」** の2つのテロップコンテナが最初から用意されています。
AIが指定した「Window ID（後述）」によって、どちらの枠が使われるかが自動的に決まります。

### ① 通常テロップ枠（normalグループ）
画面の右上に、複数のテロップを「カスケード（ずらして重ねる）」形式で表示します。テロップが出るたびにJavaScriptが動的に `.telop-item` 要素を生成します。

```html
<!-- これがページに1つ存在する「入れ物」 -->
<div id="telop-multi-container">

    <!-- テロップが出るたびに、↓のような要素がJSによって自動生成される -->
    <div class="telop-item window-variety layout-flat">

        <!-- テロップ左上の「ラベル（バッジ）」 -->
        <div class="teloop-badge">AI</div>

        <!-- テキスト本体 -->
        <div class="text-wrapper">
            <div class="topic-line">トピック（上段）</div>
            <div class="main-line">メインテキスト（下段）</div>
        </div>

    </div>

</div>
```

> **ポイント:** `.telop-item` はJSが動的に生成します。CSSで「通常テロップ全体」をターゲットにしたい場合は `#telop-multi-container .telop-item { ... }` のように書きます。

### ② 特別枠（specialグループ）
画面下部の解説テロップや、名前の紹介など、通常テロップとは「独立して（被って）」表示させたい時に使う枠です。こちらはページに最初から1つだけ固定で存在します。

```html
<!-- 特別枠の入れ物（ページに1つ固定） -->
<div id="explain-container">

    <!-- 特別枠の左上バッジ（IDで指定できる） -->
    <div id="exp-badge"></div>

    <!-- テキスト本体 -->
    <div id="exp-text-wrapper">
        <div class="topic-line" id="exp-topic">トピック（上段）</div>
        <div class="main-line"  id="exp-main">メインテキスト（下段）</div>
    </div>

</div>
```

---

## 🔍 3. CSS で使えるセレクター完全リファレンス

テーマCSSを書くときに「どのセレクターを書けばどこに効くか」を一覧で示します。

### 通常テロップ（normalグループ）のセレクター

| セレクター | 種別 | 役割 |
|---|---|---|
| `#telop-multi-container` | ID | 通常テロップ全体の入れ物。直接スタイルを当てることはほとんどない |
| `.telop-item` | クラス | 個々のテロップ要素。Window IDやLayout IDもここに付く |
| `.telop-item.visible` | クラス | テロップが表示中の状態。アニメーション制御に使う |
| `.teloop-badge` | クラス | テロップ左上の**赤いラベル（バッジ）**。ここをCSSで装飾する |
| `.teloop-badge.has-text` | クラス | バッジにテキストがある（表示すべき）状態。`display: block` で表示する |
| `.no-badge` | クラス | バッジなしのときに `.telop-item` へ付与される。上余白の調整に使う |
| `.text-wrapper` | クラス | トピックとメインテキストをまとめるラッパー |
| `.topic-line` | クラス | トピック（上段テキスト） |
| `.main-line` | クラス | メインテキスト（下段テキスト） |
| `.h1` / `.h2` | クラス | AIが強調したい語句に付ける強調クラス。色を変えるなどに使う |

#### バッジをカスタマイズする例

```css
/* 赤いバッジをオリジナルデザインにする例 */
.teloop-badge {
    /* バッジが非表示のとき（デフォルト） */
    display: none;
    position: absolute;
    top: -45px;
    left: -55px;

    /* 見た目のデザイン */
    background: #ff0000;   /* 背景色（赤） */
    color: white;
    font-size: 26px;
    font-weight: 900;
    padding: 10px 15px;
    border-radius: 8px;
    border: 3px solid white;
    transform: rotate(-5deg);  /* 少し傾ける */
    box-shadow: 4px 4px 10px rgba(0,0,0,0.5);
    white-space: nowrap;
    z-index: 10;
}

/* バッジにテキストが入ったとき（表示する） */
.teloop-badge.has-text {
    display: block;
}
```

---

### 特別枠（specialグループ）のセレクター

| セレクター | 種別 | 役割 |
|---|---|---|
| `#explain-container` | ID | 特別枠全体の入れ物。Window IDのクラスがここに付く |
| `#explain-container.visible` | ID＋クラス | 特別枠が表示中の状態。アニメーション制御に使う |
| `#exp-badge` | ID | 特別枠の**左上バッジ**（解説ウィンドウで使うラベル） |
| `#exp-badge.has-text` | ID＋クラス | バッジにテキストがある状態。`display: block` で表示する |
| `#exp-text-wrapper` | ID | 特別枠のテキストラッパー |
| `#exp-topic` | ID | 特別枠のトピック行（`.topic-line` クラスも同時に付いている） |
| `#exp-main` | ID | 特別枠のメインテキスト行（`.main-line` クラスも同時に付いている） |

> **IDとクラスの違い:** `#` で始まるIDセレクター（`#exp-badge`など）はページ内に1つしかない要素を指します。`.` で始まるクラスセレクター（`.teloop-badge`など）は複数の要素に付く可能性があります。

#### 特別枠バッジをカスタマイズする例

```css
/* 解説ウィンドウ専用バッジのデザイン例 */
.window-explain #exp-badge {
    display: none;          /* デフォルトは非表示 */
    position: absolute;
    top: -25px;
    left: 30px;
    background: #00d2ff;    /* 水色 */
    color: #000;
    font-weight: 900;
    padding: 5px 25px;
    border-radius: 20px;
    font-size: 24px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.3);
}

.window-explain #exp-badge.has-text {
    display: block;         /* テキストがあれば表示 */
}
```

---

### 画像枠のセレクター

| セレクター | 種別 | 役割 |
|---|---|---|
| `#img-container` | ID | 画像表示の入れ物。Window IDのクラスがここに付く |
| `#img-container.visible` | ID＋クラス | 画像が表示中の状態 |

---

## 🪪 4. 既存の Window ID 一覧とシステム制御

TeloPonのAIには、あらかじめ決められた「Window ID」を指定させます。CSSでは、この指定されたWindow IDのクラス（`.window-xxx`）に対してデザインやシステムへの命令を記述します。

### 📌 通常枠（normalグループ）のID
これらは `.telop-item` に追加され、UIの「通常枠表示(秒)」の設定時間で消えます。

* **`.window-simple`** : 最も標準的でシンプルなテロップ用。
* **`.window-variety`** : バラエティ番組風のポップなデザイン用。
* **`.window-news`** : ニュース番組風の堅いデザイン用。
* **`.window-cute`** : 手書き風や可愛らしいデザイン用。
* **`.window-digital`** : サイバー感のあるデジタルなデザイン用。
* **`.window-reply`** : 視聴者への返信やメンション（○○さんへ！）など、目立たせる横幅の広いデザイン用。

### 📌 特別枠（specialグループ）のID
これらは `#explain-container` に適用されます。システムに「これは特別枠だよ」と教えるために、**CSS内で必ず `--window-group: special;` を指定する必要があります。**

* **`.window-explain`** : 長文の解説や、補足情報を画面下部にどっしり出す用途。
* **`.window-nameplate`** : TV番組風の「二つ名＋名前」をデカデカと出す用途。

---

## 🔊 5. 音を鳴らす・システムを制御する（CSS変数）

TeloPonの最大の特徴は、**「CSSからシステム（PythonやJS側）に設定を逆注入できる」**点です。これを「CSS変数（`--` から始まる記述）」で行います。

### 💡 効果音を鳴らす（`--sound-file`）
テロップが表示された瞬間に効果音を鳴らしたい場合、CSS内に `--sound-file` を記述します。音声ファイルは `TeloPon/sounds/` フォルダにあるものを指定します。

```css
/* バラエティ枠が出た瞬間、「ポンッ」という音を鳴らす例 */
.window-variety {
    background: linear-gradient(135deg, #ff00cc, #333399);
    color: white;
    /* TeloPon/sounds/sound1.mp3 を再生するようシステムに指示 */
    --sound-file: "sound1.mp3";
}
```

### 💡 グループを「特別枠」に変更する（`--window-group`）
前述の通り、`window-explain` などの特別枠デザインを作るときは、この変数を指定しないと通常枠扱いになってしまいます。

```css
.window-explain {
    position: absolute;
    bottom: 40px;
    /* これを付けることで「#explain-container」で描画されるようになる */
    --window-group: special;
    --sound-file: "sound2.mp3";
}
```

### 💡 表示時間を強制上書きする（`--display-duration`）
UI画面で設定された秒数（例：5秒）を無視して、「このテロップだけは絶対に10秒間表示する」といった強制指定が可能です。

```css
.window-nameplate {
    /* UIの設定を無視して、名前窓は強制的に5秒で消す */
    --window-group: special;
    --display-duration: 5;
}
```

---

## 📐 6. Layout ID（文字バランスの調整）

通常枠（`.telop-item`）には、Window IDに加えて「Layout ID」も付与されます。これを使ってトピックとメインテキストの「文字の大きさのバランス」を調整します。

* **`.layout-flat`** : トピックとメインテキストを同じ大きさにするフラットなレイアウト。
* **`.layout-big-top`** : 上段（トピック）の文字を大きく、下段（メイン）を小さくするレイアウト。
* **`.layout-big-bottom`** : 上段（トピック）の文字を小さく、下段（メイン）を大きく強調するレイアウト。

**CSSでの活用例:**
```css
/* flatが指定されたら、元のデザインの枠線などを消して文字だけに特化させる等の例外処理も可能 */
.window-variety.layout-flat .topic-line {
    background: none !important;
    font-size: 36px !important;
}
```

---

## 🎬 7. アニメーションの実装（`.visible`）

テロップが画面に出現する際、システムはコンテナに対して **`.visible` クラス** を付与します。
これを利用して、CSSの `transition` や `transform` を使った滑らかな登場アニメーションを作ります。

### 通常テロップのアニメーション
通常テロップ（`.telop-item`）のベースアニメーション（フワッと拡大して登場）は `base.html` 内に固定で定義されています。テーマCSSで上書きしたい場合は、より詳細なセレクターで記述します。

```css
/* 例：通常テロップを「右からスライドイン」に変更する */
#telop-multi-container .telop-item {
    opacity: 0;
    transform: translateX(80px);   /* 右にずれた状態から */
    transition: opacity 0.4s, transform 0.4s;
}
#telop-multi-container .telop-item.visible {
    opacity: 1;
    transform: translateX(0);      /* 定位置に滑り込む */
}
```

### 特別枠のアニメーション
特別枠（`#explain-container`）のアニメーションはテーマCSSで完全にコントロールします。Window IDのクラスに `opacity` と `transform` を設定し、`.visible` が付いたときに変化させます。

```css
/* =========================================
   解説テロップ（下からスライドして登場）
   ========================================= */
.window-explain {
    position: absolute; left: 40px; bottom: 40px;
    opacity: 0;                          /* 初期状態：透明 */
    transform: translateY(30px);         /* 初期状態：少し下にずれている */
    transition: all 0.6s cubic-bezier(0.2, 0.8, 0.2, 1);
    --window-group: special;
}
.window-explain.visible {
    opacity: 1;                          /* 表示：不透明に */
    transform: translateY(0);            /* 表示：定位置に移動 */
}
```

---

## 💡 8. 開発時のコツ（デバッグ方法）

テーマのCSSを調整する際、毎回AIに喋らせて確認するのは大変です。
デザインを確認・調整したい時は、TeloPonをデバッグモード（`-d`）で起動してください。

```bash
python telopon_live.py -d
```

UIの下部に**「手動操作パネル」**が表示されます。
そこにトピックやメインテキスト、確認したいウィンドウIDを入力して「強制表示」ボタンを押すことで、AIを介さずに何度でもテロップの描画テスト（音の再生確認含む）を行うことができます。
OBSのブラウザソース上でリアルタイムにCSSの変更を確認しながらデザインを仕上げましょう！
