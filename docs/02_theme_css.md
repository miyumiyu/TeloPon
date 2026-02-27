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
画面の右上など、次々と順番に表示される標準的なテロップ枠です。
```html
<div id="teloop-container" class="ここにWindowIDとLayoutIDが入る">
    <div id="teloop-badge"></div>
    <div id="text-wrapper">
        <div class="topic-line">トピック</div>
        <div class="main-line">メインテキスト</div>
    </div>
</div>
```

### ② 特別枠（specialグループ）
画面下部の解説テロップや、名前の紹介など、通常テロップとは「独立して（被って）」表示させたい時に使う枠です。
```html
<div id="explain-container" class="ここにWindowIDが入る">
    <div id="exp-badge"></div>
    <div id="exp-text-wrapper">
        <div class="topic-line">トピック</div>
        <div class="main-line">メインテキスト</div>
    </div>
</div>
```

---

## 🪪 3. 既存の Window ID 一覧とシステム制御

TeloPonのAIには、あらかじめ決められた「Window ID」を指定させます。CSSでは、この指定されたWindow IDのクラス（`.window-xxx`）に対してデザインやシステムへの命令を記述します。

### 📌 通常枠（normalグループ）のID
これらは `#teloop-container` に適用され、UIの「通常枠表示(秒)」の設定時間で消えます。

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

## 🔊 4. 音を鳴らす・システムを制御する（CSS変数）

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

## 📐 5. Layout ID（文字バランスの調整）

通常枠（`#teloop-container`）には、Window IDに加えて「Layout ID」も付与されます。これを使ってトピックとメインテキストの「文字の大きさのバランス」を調整します。

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

## 🎬 6. アニメーションの実装（`.visible`）

テロップが画面に出現する際、システムはコンテナに対して **`.visible` クラス** を付与します。
これを利用して、CSSの `transition` や `transform` を使った滑らかな登場アニメーションを作ります。

```css
/* =========================================
   通常テロップ（右上からフワッと拡大して登場）
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
   解説テロップ（下からスライドして登場）
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

## 💡 7. 開発時のコツ（デバッグ方法）

テーマのCSSを調整する際、毎回AIに喋らせて確認するのは大変です。
デザインを確認・調整したい時は、TeloPonをデバッグモード（`-d`）で起動してください。

```bash
python telopon_live.py -d
```

UIの下部に**「手動操作パネル」**が表示されます。
そこにトピックやメインテキスト、確認したいウィンドウIDを入力して「強制表示」ボタンを押すことで、AIを介さずに何度でもテロップの描画テスト（音の再生確認含む）を行うことができます。
OBSのブラウザソース上でリアルタイムにCSSの変更を確認しながらデザインを仕上げましょう！
