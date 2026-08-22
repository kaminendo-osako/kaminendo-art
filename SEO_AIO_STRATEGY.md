# 株式会社かみねんど SEO・AIO 実装計画

> **前提の補正（重要）**
> 調査A・Bは `index.html` が 36,880 bytes だった時点のスナップショットです。**実測したところ、ローカルの `C:\RYUフォルダ\021_HP制作\KAMINENDO\index.html` は 45,456 bytes に更新されており、Phase 1 相当の一部が既に適用済み（未コミット）** でした。
> 既適用：title/description 刷新・`canonical`・`robots` メタ・OGP/Twitter フルセット（`assets/ogp.png` 1200×630 実在）・JSON-LD `@graph`（Organization / Person×2 / WebSite / WebPage、JSONパース検証済み）・`robots.txt`・`sitemap.xml`。
> `git status` は `M index.html` / `?? assets/ogp.png` / `?? robots.txt` / `?? sitemap.xml` の未コミット状態。**本番 kaminendo.art にはまだ反映されていません**（Bの外部実測が旧状態を返しているのはこのため）。
> 以下は「既適用分を前提に、その先を作る計画」です。

---

## 1. 現状評価（何ができていて、何が致命的に欠けているか）

### 1-1. 領域別スコア（10段階／2026-08-22 ローカル実測時点）

| 領域 | 点 | できていること | 欠けていること |
|---|---|---|---|
| クロール基盤 | **6** | robots.txt（AI検索bot含め全許可）・sitemap.xml・canonical・robotsメタ | **未pushで本番未反映**／404.html なし／www のHTTPS証明書エラー／`l-os.jp` は robots.txt・sitemap とも404 |
| 構造化データ | **7** | Organization（法人番号・設立日・住所・OfferCatalog 4事業）＋ Person×2 ＋ WebSite ＋ WebPage。JSON妥当 | `sameAs` が LINE 1本だけ／`telephone` なし／下層ページが無いので BreadcrumbList・ProfilePage が未使用 |
| head / OGP | **8** | canonical・og:image(1200×630)・twitter:card・author・robots すべて実在 | GSC検証メタ／GA4／apple-touch-icon が無い |
| キーワード適合 | **3** | title・description に「LINE公式アカウントの構築・運用・開発」が入った（大改善） | **h2 4本がすべてひらがな＋語中スペースでキーワード価値ゼロ**／h1 は社名のみ／地域語ゼロ／「LINE公式アカウント」の本文出現が実質1回 |
| コンテンツ量・受け皿 | **1** | 会社概要6項目の事実密度は高い | **静的本文 約1,400字／クロール可能URLが2つだけ**。料金・実績・事例・FAQ・業種別・地域別が全部ゼロ。狙うKWの着地点が物理的に存在しない |
| E-E-A-T／エンティティ | **3** | 法人番号・設立日・代表実名・所在地が平文HTMLにある（AIOに最も効く資産） | SNS等の外部プロフィールが1つもない＝`sameAs` が張れない／実績ゼロ／電話番号なし／**会社としてのプライバシーポリシーが存在しない**（フォームで氏名・メール・相談内容を収集しGASへPOSTしているのに） |
| 外部シグナル | **1** | — | 被リンク・言及ともほぼゼロ。**`l-os.jp` へのリンクが会社サイトに1本も無い**。PR TIMES・note・ITreview・BOXIL いずれも未着手 |
| AI引用可能性 | **2** | 構造化データは整った | 引用の原資（数字・定義文・料金・事例）が無い。順位が付いていないので AIO 引用の前提を満たさない |
| 表示速度／CWV | **3** | 外部依存は Google Fonts のみ・`display=swap`・preconnect 済み | 全画面オープニングがクリックまで本文を1px も見せない（実質4秒超）／ロゴに width/height なし（CLS）／`setInterval` が clear されず永続 |
| アクセシビリティ | **2** | ランドマーク・`label for` は正しい | オープニングがクリック解除のみ（キーボード到達不可の恐れ）／canvas 3つに代替テキストなし／`prefers-reduced-motion` 未考慮 |
| 計測 | **0** | — | GSC・Bing WMT・GA4 すべて未登録。**現状の流入はデータが存在しない** |

**総合：3 / 10**（技術の土台だけ7点相当まで上がったが、その上に載せる中身と外部シグナルが1点台）

### 1-2. 致命的な欠落（この順で潰す）

1. **`l-os.jp` に `noindex` もサービスLPも無い** — テナント運用画面 `/t/<key>` と `/admin` が野ざらしのままインデックスされうる（事故側のリスク）。同時に、自社最大の武器である L OS が検索上に存在しない（機会側の損失）。**リスクと機会が同じ1施策で解ける最優先項目。**
2. **受け皿URLが2つしかない** — どのキーワードも着地できない。他の全施策の前提条件。
3. **計測ゼロ** — 改善したかどうかを判定する手段が無い。
4. **`privacy.html` が別アプリ（THE WORLD SCREEN）の規約** — 問い合わせフォームを持つ企業サイトとして是正必須。
5. **www の HTTPS 証明書エラー** — 名刺・SNSで www 付き表記をされると到達不能。
6. **h2 全滅** — デザイン意図なので壊さず解く（§5）。
7. **拠点の矛盾（サイト＝東京都葛飾区／依頼文＝名古屋近郊）** — 地域KW・GBP・NAP統一の全てが変わる。**着手前に確定が必要（§9-Q1）。**

---

## 2. 基本方針

**SEO：ヘッド語は最初から捨て、「受け皿を作る」ことと「指名検索の100%確保」に投資する。**
「LINE公式アカウント 運用代行」級のヘッド語は1.2〜1.5万字の比較メディアが占有しており、設立2026年3月・被リンクほぼゼロのドメインで正面から競うのは投資の無駄です。したがって狙いは3つに限定します。(1)**指名検索の完全確保**＝`株式会社かみねんど` `大迫龍太` `L OS` で1位を取り切る（現在これすら取れていない。「かみねんど」という語が nendo・紙粘土・pixiv絵師に強く占有されているため、Organization 構造化＋外部言及で「株式会社かみねんど」完全一致のエンティティを立てる必要がある）。(2)**不満クエリの刈り取り**＝`Lステップ 高い／代替／乗り換え／解約` は競合が薄く、自社SaaSを持つ構築会社という立場でしか書けない領域。(3)**地域×業種×課題のロングテール**＝1語あたりの需要は極小だが競合ページが物理的に存在しない区画を20〜30本積む。そのすべての前提として、まず kaminendo.art をマルチページ化し、`l-os.jp` に製品LPを公開します。

**AIO：AIO専用の小細工はやらない。「順位」「静的HTML」「引用できる事実」の3点に集約する。**
日本国内実測でAI Overview引用の90.2%が検索10位以内から来ており、AIO対策の本体は通常SEOです。加えて Vercel×MERJ の実測で主要AIクローラ（GPTBot / OAI-SearchBot / ClaudeBot / PerplexityBot）はJavaScriptを実行しないため、**JSでしか出ないテキストはAIから完全に不可視**（現状ヒーローのビジョン文「クリエイティブで 心を豊かにする。」がまさにこれで、titleの最重要コピーが静的HTMLに無い）。構造化データはAhrefsの対照実験でAI引用増加が否定されているので「Google のリッチリザルトとエンティティ明示のための安価な土台」以上には期待せず、llms.txt は「5分で作れて害はないが期待しない」最下位扱いにします。実際に効く投資は、**可視HTMLに数字・定義文・更新日を書くこと**と、**日本語圏でAIが実際に引用しているドメイン（PR TIMES・note・ITreview/BOXIL）に自社を載せること**の2つです。

---

## 3. 狙うキーワード設計

> 検索ボリュームの実数は未取得（キーワードプランナー／ラッコキーワードでの実測が別途必要）。需要は SERP 構成からの相対的見立て。

### 3-1. 軸キーワード（各ページ1本だけ背負わせる）

| # | 軸KW | 着地ページ | 需要 | 競合 | 勝算 | 根拠 |
|---|---|---|---|---|---|---|
| 1 | 株式会社かみねんど | `/`（トップ） | 小 | 低 | ◎ | 指名。ただし同名語に占有されているため Organization＋外部言及が要る |
| 2 | L OS / LINE 運用 SaaS | `l-os.jp/`（製品LP） | 小→中 | 低 | ◎ | 現在ログイン画面のみ＝完全な空白 |
| 3 | Lステップ 代替 | `l-os.jp/compare/lstep` | 中 | 中 | ○ | 顕在不満層。自社SaaS保有が唯一の差別化 |
| 4 | LINE公式アカウント 構築代行 費用 | `/line/price/` | 中 | 高 | △→○ | **実額開示が条件**。競合は全社「幅」でしか書いていない |
| 5 | LINE公式アカウント 運用代行 ＋地域 | `/line/[地域]/` | 小 | 中 | ○ | 拠点確定後。名古屋は寡占されておらず入れる |
| 6 | 展示会 ブース 集客 マジシャン | `/performer/exhibition/` | 小 | **低** | ◎ | ほぼ空白かつBtoB単価が高い。最推奨 |
| 7 | 忘年会 余興 マジシャン ＋地域 | `/performer/year-end-party/` | 中（季節） | 中 | ○ | 9〜11月集中。8月着手で間に合う |
| 8 | 大迫龍太 マジシャン | `/members/ryu.html` | 小 | 低 | ◎ | E-E-A-Tの受け皿。既存 `osako.kaminendo.art` と相互リンク |
| 9 | リッチメニュー 作り方 / サイズ | `l-os.jp/blog/richmenu-*` | **大** | 高 | △ | 自社に確定描画エンジンがある数少ない大需要語 |
| 10 | IT導入補助金 LINE公式アカウント | `/line/subsidy/` | 小〜中 | 低〜中 | ○ | 価格離脱層を拾う高単価導線。※IT導入支援事業者の登録有無で書ける内容が変わる（要確認） |

### 3-2. ロングテール（型別・20〜30本を月2〜3本ペースで）

| 型 | 例 | 競合 | 備考 |
|---|---|---|---|
| 不満クエリ | `Lステップ 高い` `Lステップ 乗り換え` `Lステップ 解約 方法` `Lステップ 構築代行 失敗` | 低〜中 | **最優先。ほぼ空白地帯** |
| 地域×業種×課題 | `LINE公式アカウント 構築 名古屋 美容室` `公式LINE 予約 自動化 整体院 愛知` `LINE 集客 学習塾 東海` `LINE運用代行 工務店 岐阜` | 低 | 競合ページが物理的に無い。合算で意味を持つ |
| 価格・比較 | `LINE公式アカウント 構築 安い` `公式LINE 構築代行 相場 いくら` `L OS Lステップ エルメ 比較` | 中 | 料金実額の公開が前提 |
| マジシャン×BtoB | `販促イベント パフォーマー 手配` `商業施設 マジックショー 手配` `テーブルマジック 派遣 費用` | **低** | 競合ほぼ皆無 |
| かみねんど固有 | `パフォーマー 集客 LINE` `イベント会社 顧客管理 LINE` `出演依頼 受付 自動化` | **ゼロ** | 代表がマジシャン兼経営者という事実が説得力そのもの。①と②を接続する唯一の導線 |

### 3-3. AI検索での想定質問（このまま h2 と FAQ の設問文に使う）

| # | 想定質問（日本語そのまま） | 答えるページ | 引用されるために必要なもの |
|---|---|---|---|
| Q1 | LINE公式アカウントの構築代行はいくらかかりますか？ | `/line/price/` | **実額のプラン表**（幅ではなく数字） |
| Q2 | Lステップより安いLINE拡張ツールはありますか？ | `l-os.jp/compare/lstep` | 3列比較表（L OS／Lステップ／エルメ）＋実額 |
| Q3 | L OS とは何ですか？ | `l-os.jp/` | **「L OS とは、〜です。」の定義文1文**を冒頭に |
| Q4 | LINE公式アカウントの運用代行を名古屋（東京）で頼めますか？ | `/line/[地域]/` | 地域名＋対応範囲＋NAP一致 |
| Q5 | 株式会社かみねんどはどんな会社ですか？ | `/company/` | 法人番号・設立日・代表者・所在地の平文表示（**既にある強み**） |
| Q6 | 展示会のブースに集客するにはどうすればいいですか？ | `/performer/exhibition/` | 冒頭2文で結論＋具体的な進め方 |
| Q7 | 忘年会の余興でマジシャンを呼ぶ費用の相場は？ | `/performer/year-end-party/` | 実額（要確認） |
| Q8 | IT導入補助金でLINE公式アカウントは対象になりますか？ | `/line/subsidy/` | 枠・上限額・条件の明記 |
| Q9 | 大迫龍太とはどんな人ですか？ | `/members/ryu.html` | 肩書・経歴・出演先・外部プロフィールへのリンク |
| Q10 | LINEのリッチメニューは自分で作れますか？ | `l-os.jp/blog/richmenu-*` | 手順＋サイズ表＋画像 |

---

## 4. 今すぐやる技術対策（ファイル単位・そのまま貼れる形）

### 4-0. 既に入っているもの（再実装しない）

`index.html` の 6〜31行（title/description/canonical/robots/author/OGP/Twitter）、34〜181行（JSON-LD `@graph`）、`robots.txt`、`sitemap.xml`、`assets/ogp.png`。
**robots.txt は現状の内容のままで妥当です**（検索・回答系bot全許可＋Sitemap宣言）。1点だけ、`ChatGPT-User` と `Perplexity-User` は各社公式が「robots.txt に従わない／概ね無視する」と明記しているため、許可記述は実質的に無効である旨をコメントで残しておくと運用時に誤解しません。

---

### 4-1. 【新規】`C:\RYUフォルダ\021_HP制作\KAMINENDO\404.html`

GitHub Pages はリポジトリ直下の `404.html` を自動で使います。現状は英語の GitHub 既定ページが出ています。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ページが見つかりません｜株式会社かみねんど</title>
<meta name="robots" content="noindex,follow">
<meta name="theme-color" content="#05070f">
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'%3E%3Ctext y='.9em' font-size='90'%3E%E2%9C%A6%3C/text%3E%3C/svg%3E">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=DotGothic16&display=swap" rel="stylesheet">
<style>
:root{--ink:#e8ecf5;--bg:#05070f;--cyan:#9fe8ff;--green:#8ef0b4}
*{margin:0;padding:0;box-sizing:border-box}
body{background:var(--bg);color:var(--ink);font-family:'DotGothic16','MS Gothic',monospace;
     line-height:1.9;min-height:100vh;display:grid;place-items:center;padding:24px}
.win{background:rgba(4,8,18,.92);border:3px solid #fff;border-radius:8px;padding:28px 22px;max-width:560px;width:100%}
.cap{display:block;font-size:11px;letter-spacing:.4em;color:var(--green);margin-bottom:10px}
h1{font-size:clamp(20px,6vw,28px);margin-bottom:14px}
p{font-size:14px;color:#c5cede}
.btn{display:inline-block;margin-top:20px;padding:10px 18px;border:2px solid var(--cyan);
     border-radius:6px;color:var(--cyan);text-decoration:none;font-size:14px}
</style>
</head>
<body>
<div class="win">
  <span class="cap">404 NOT FOUND</span>
  <h1>この さきは みちが ない</h1>
  <p>お探しのページは 見つかりませんでした。<br>URLが変わったか、削除された可能性があります。</p>
  <a class="btn" href="https://kaminendo.art/">▶ トップへもどる</a>
</div>
</body>
</html>
```

---

### 4-2. 【編集】`index.html` head — 追記する3行

31行（Google Fonts の `<link>`）の直後に挿入。

```html
<link rel="apple-touch-icon" href="/assets/ogp.png">
<!-- Search Console 所有権確認: GSCで発行される content 値に差し替えること（要取得） -->
<meta name="google-site-verification" content="【要取得：GSCが発行する値】">
```

GA4 は測定ID確定後に `</head>` 直前へ。**測定IDは未発行（要確認）。**

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-【要取得】"></script>
<script>
window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)}
gtag('js',new Date());gtag('config','G-【要取得】');
</script>
```

---

### 4-3. 【編集】`index.html` JSON-LD — 3か所を差し替え／追記

**(a) `sameAs` の拡充（80行）** — 現状 LINE 1本のみ。エンティティ同定の要。

```json
      "sameAs": [
        "https://line.me/R/ti/p/@687czsmv",
        "https://osako.kaminendo.art/",
        "https://fukui.kaminendo.art/",
        "https://l-os.jp/"
      ],
```

> **要確認**：法人番号公表サイト（国税庁）／gBizINFO の自社ページURL、および X・Instagram・YouTube・note の公式アカウント。**存在を目視確認できたURLだけを追加すること。実在しないURLを `sameAs` に書くのは逆効果。**

**(b) `telephone` の追加（52行 `email` の直後）** — **電話番号を公開する経営判断が出てからのみ**。

```json
      "telephone": "【要確認：公開する電話番号／公開しないならこの行ごと入れない】",
```

**(c) `LocalBusiness` は入れない（判断）**
かみねんどは来店型ではなく、GBPの営業時間・経路・口コミが機能しません。`LocalBusiness` を入れると Google がGBPとの不一致を検出しうるため、**`Organization` ＋ `PostalAddress`（実装済み）で止めるのが正解**です。GBP を開設する判断が出た時点で初めて `LocalBusiness` に格上げします。

---

### 4-4. 【編集】`index.html` 本文 — JS依存テキストの静的化ほか（AIクローラ対策の本丸）

**(1) ヒーローのビジョン文を静的HTMLへ（439行）**

```html
<p class="hero-vision" id="vision">クリエイティブで 心を豊かにする。</p>
```

**同時に 635〜639行のタイプライタを1行修正**（そのままだと静的テキストに追記されて二重になります）。`var vTimer = setInterval(...)` の直前に：

```js
    vision.textContent = '';
```

**(2) ヒーロー本文に事業語を入れる（440〜441行）** — h1 が社名のみなので、直下の可視テキストで補います。

```html
      <p class="hero-copy">株式会社かみねんどは、<b>LINE公式アカウントの構築・運用・開発</b>を行う会社です。マジシャンなどのパフォーマー派遣、商品開発・販売、経営コンサルティングも手がけています。</p>
      <p class="hero-copy" style="margin-top:6px">発想力・技術力・演出力を かけあわせて、あたらしい価値をつくる。マジックの「驚き」と テクノロジーの「仕組み」で、あなたのビジネスを進化させます。</p>
```

**(3) 会社概要テーブルに3行追加（523行の後）** — 事実密度の高い区画をさらに強化。

```html
        <tr><th>連絡先</th><td>osako@kaminendo.art</td></tr>
        <tr><th>公式サイト</th><td>https://kaminendo.art/</td></tr>
        <tr><th>提供サービス</th><td>LINE公式アカウント運用システム「L OS」（<a href="https://l-os.jp/" target="_blank" rel="noopener">l-os.jp</a>）</td></tr>
```

**(4) フッターに内部リンクを追加（557行の後）** — privacy が孤立ページである問題と、`l-os.jp` が会社サイトから完全に切り離されている問題を同時に解消。

```html
  <p style="margin-top:6px">
    <a href="https://l-os.jp/" target="_blank" rel="noopener">L OS（LINE運用システム）</a>
    ｜<a href="https://fukui.kaminendo.art/" target="_blank" rel="noopener">FUKUI QUEST</a>
    ｜<a href="/privacy-policy.html">プライバシーポリシー</a>
  </p>
```

**(5) ロゴに寸法と遅延指定（422行）** — CLS対策。

```html
  <a class="brand" href="#top"><img src="assets/logo_white.png" alt="株式会社かみねんど" width="585" height="120" decoding="async"></a>
```

**(6) canvas 3つに代替テキスト** — 493 / 502 / 695行付近。

```html
      <canvas id="ceo-face" width="16" height="18" role="img" aria-label="代表取締役 大迫龍太のドット絵"></canvas>
      <canvas id="fukui-face" width="16" height="18" role="img" aria-label="執行役員 福井創祈のドット絵"></canvas>
      <!-- #chara は装飾なので逆に隠す -->
      <canvas id="chara" aria-hidden="true"></canvas>
```

**(7) `prefers-reduced-motion`（`</style>` 直前 = 416行の前）**

```css
@media (prefers-reduced-motion: reduce){
  *,*::before,*::after{animation-duration:.01ms!important;animation-iteration-count:1!important;
    transition-duration:.01ms!important;scroll-behavior:auto!important}
  #stars i{animation:none;opacity:.35}
  .pop{opacity:1!important;transform:none!important}
}
```

**(8) オープニングにキーボード解除（947行 `op.addEventListener('click', ...)` の近傍）** — 現状クリックのみ＝キーボード利用者が本文に到達できない恐れ。

```js
  document.addEventListener('keydown', function(e){
    if(e.key==='Enter'||e.key===' '||e.key==='Escape'){ op.click(); }
  });
```

---

### 4-5. 【新規】`privacy-policy.html`（会社としての個人情報保護方針）

> **URL設計の判断**：既存 `/privacy.html` は THE WORLD SCREEN のもので、**App Store にこのURLが登録されている可能性が高い**（変更・移動は審査リスク）。したがって **`/privacy.html` は据え置き**、会社用を **`/privacy-policy.html`** に新設します（自然言語スラッグとしても妥当）。App Store 登録URLが確認できたら `/apps/the-world-screen/privacy.html` への整理を検討（§9-Q6）。

以下は骨子。**事実の創作を避けるため、確定していない項目は `【要確認】` のまま残しています。** 記載内容はすべて現行 `index.html` の実装（氏名・メールアドレス・ご相談内容を収集し `script.google.com` の GAS エンドポイントへPOST、返信先は `osako@kaminendo.art`）から確認した事実のみです。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>プライバシーポリシー｜株式会社かみねんど</title>
<meta name="description" content="株式会社かみねんどの個人情報保護方針。お問い合わせフォームで取得する情報の利用目的、第三者提供、開示請求の窓口について記載しています。">
<link rel="canonical" href="https://kaminendo.art/privacy-policy.html">
<meta name="robots" content="index,follow">
<!-- ▼ index.html と同じ <style> ブロックを流用（デザイン統一）。将来は assets/rpg.css に外部化 -->
</head>
<body>
<main>
  <h1>プライバシーポリシー</h1>
  <p>最終更新日：2026年8月22日</p>

  <h2>1. 事業者情報</h2>
  <p>株式会社かみねんど（法人番号 2011801047648）<br>
     東京都葛飾区小菅4丁目21-38<br>
     代表取締役　大迫 龍太<br>
     連絡先：osako@kaminendo.art</p>

  <h2>2. 取得する個人情報</h2>
  <p>当社ウェブサイト（kaminendo.art）のお問い合わせフォームでは、お名前・メールアドレス・ご相談内容を取得します。</p>

  <h2>3. 利用目的</h2>
  <ul>
    <li>お問い合わせへの回答、お見積りのご提示</li>
    <li>ご依頼いただいたサービスの提供および連絡</li>
  </ul>

  <h2>4. 第三者提供・外部サービスの利用</h2>
  <p>ご本人の同意なく第三者へ提供することはありません。ただし、フォーム送信内容の受け取りに Google LLC の提供するサービス（Google Apps Script／Google スプレッドシート）を利用しており、同社のサーバー上に保管されます。【要確認：アクセス解析（Google アナリティクス）導入後は本項に追記】</p>

  <h2>5. 保存期間</h2>
  <p>【要確認：お問い合わせ内容の保存期間（例：最終連絡から3年）】</p>

  <h2>6. 開示・訂正・削除のご請求</h2>
  <p>ご本人からのご請求に応じ、法令に基づき対応します。osako@kaminendo.art までご連絡ください。</p>

  <h2>7. LINE公式アカウントについて</h2>
  <p>【要確認：公式LINEで友だち登録者の情報をどこまで取得・利用しているか。運用実態にあわせて記載】</p>

  <h2>8. 改定</h2>
  <p>本方針の内容は、必要に応じて改定することがあります。</p>

  <p><a href="/">▶ トップへもどる</a></p>
</main>
</body>
</html>
```

**あわせて既存 `privacy.html` に2点だけ追記**（別プロダクトの規約が会社ドメインに同居していること自体は残す前提で、責任主体を明示する）：

```html
<title>プライバシーポリシー（THE WORLD SCREEN）｜株式会社かみねんど</title>
<meta name="description" content="iOSアプリ「THE WORLD SCREEN」の個人情報の取り扱いについて。提供：株式会社かみねんど。">
<link rel="canonical" href="https://kaminendo.art/privacy.html">
<!-- 本文末尾に -->
<p>本ポリシーはiOSアプリ「THE WORLD SCREEN」に関するものです。株式会社かみねんど（会社）としてのプライバシーポリシーは<a href="/privacy-policy.html">こちら</a>をご覧ください。</p>
```

---

### 4-6. 【更新】`sitemap.xml`（ページ追加後の完成形）

追加したページを **公開したものだけ** 書きます（存在しないURLを書くとクロール品質が落ちます）。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://kaminendo.art/</loc><lastmod>2026-08-22</lastmod><changefreq>weekly</changefreq><priority>1.0</priority></url>
  <url><loc>https://kaminendo.art/line/</loc><lastmod>2026-08-22</lastmod><changefreq>monthly</changefreq><priority>0.9</priority></url>
  <url><loc>https://kaminendo.art/line/price.html</loc><lastmod>2026-08-22</lastmod><changefreq>monthly</changefreq><priority>0.9</priority></url>
  <url><loc>https://kaminendo.art/performer/</loc><lastmod>2026-08-22</lastmod><changefreq>monthly</changefreq><priority>0.8</priority></url>
  <url><loc>https://kaminendo.art/performer/exhibition.html</loc><lastmod>2026-08-22</lastmod><changefreq>monthly</changefreq><priority>0.8</priority></url>
  <url><loc>https://kaminendo.art/company/</loc><lastmod>2026-08-22</lastmod><changefreq>yearly</changefreq><priority>0.7</priority></url>
  <url><loc>https://kaminendo.art/members/ryu.html</loc><lastmod>2026-08-22</lastmod><changefreq>yearly</changefreq><priority>0.6</priority></url>
  <url><loc>https://kaminendo.art/privacy-policy.html</loc><lastmod>2026-08-22</lastmod><changefreq>yearly</changefreq><priority>0.3</priority></url>
  <url><loc>https://kaminendo.art/privacy.html</loc><lastmod>2026-07-27</lastmod><changefreq>yearly</changefreq><priority>0.2</priority></url>
</urlset>
```

---

### 4-7. 【新規】`llms.txt`（優先度は最下位。5分で作れて害はない、以上の期待をしない）

Ahrefs の137,000ドメイン実測で97%が0リクエスト、リクエストがあった分の96%も非AIボットです。**AI検索での引用増を期待して作るものではありません。**

```markdown
# 株式会社かみねんど

> LINE公式アカウントの構築・運用・開発を行う会社（法人番号 2011801047648／2026年3月31日設立）。
> 自社開発のLINE運用システム「L OS」を提供。マジシャンなどのパフォーマー派遣、商品開発・販売、
> 経営コンサルティングも手がける。代表は現役マジシャンの大迫龍太。

## サービス
- [LINE公式アカウントの構築・運用・開発](https://kaminendo.art/line/): 新規構築・運用支援・改善サポート・機能開発。無料アカウント診断あり
- [料金](https://kaminendo.art/line/price.html): プランと費用
- [パフォーマー派遣](https://kaminendo.art/performer/): マジシャン・パフォーマーの派遣と演出設計
- [L OS](https://l-os.jp/): 業界を問わないLINE公式アカウント運用システム

## 会社情報
- [会社概要](https://kaminendo.art/company/): 社名・所在地・代表者・設立・法人番号
- [代表 大迫龍太](https://kaminendo.art/members/ryu.html): 代表取締役／マジシャン
- [プライバシーポリシー](https://kaminendo.art/privacy-policy.html)

## 連絡先
- メール: osako@kaminendo.art
- LINE: https://line.me/R/ti/p/@687czsmv
```

---

### 4-8. 【最優先・別リポジトリ】`l-os.jp`（`line-os/`）の技術対策

**リスク側（今日中にやる価値がある）**：現状 robots.txt が404、`X-Robots-Tag` も無く、テナント運用画面がインデックスされうる状態です。

`line-os/app/robots.ts`（Next.js 16 App Router）

```ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        // 運用者コンソール・管理画面・APIはインデックスさせない
        disallow: ['/t/', '/admin', '/api/', '/login'],
      },
    ],
    sitemap: 'https://l-os.jp/sitemap.xml',
    host: 'https://l-os.jp',
  }
}
```

`line-os/app/sitemap.ts`

```ts
import type { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  const base = 'https://l-os.jp'
  const now = new Date()
  return [
    { url: `${base}/`,                changeFrequency: 'weekly',  priority: 1.0, lastModified: now },
    { url: `${base}/features`,        changeFrequency: 'monthly', priority: 0.9, lastModified: now },
    { url: `${base}/pricing`,         changeFrequency: 'monthly', priority: 0.9, lastModified: now },
    { url: `${base}/compare/lstep`,   changeFrequency: 'monthly', priority: 0.8, lastModified: now },
  ]
}
```

**`/t/[tenant]` と `/admin` のレイアウトに `noindex` を明示**（robots.txt はURLの秘匿ではないため二重に）。`line-os/app/t/[tenant]/layout.tsx` と `line-os/app/admin/layout.tsx` に：

```ts
export const metadata = { robots: { index: false, follow: false } }
```

**機会側**：`l-os.jp/` を製品LPにする（現在はログイン画面のみ）。ログイン導線を `/login` に移すか、`/` にLPを置きつつ既存ログインボタンを残すかは既存運用者への影響があるため**要判断（§9-Q5）**。LPは Server Component で静的レンダリング必須（AIクローラはJSを実行しないため、機能説明をクライアントfetchで差し込む実装にしないこと）。

---

## 5. デザインを壊さずに検索意図を満たす方法（ひらがな見出し問題の具体解）

**前提の切り分け**：ひらがな見出しはブランドそのものなので**触りません**。解くべきは「h2に検索語が1つも無い」という事実だけです。RPG風UIは既に「小さい英字ラベル（`.sec-label` = `QUEST LIST` 等、11px・letter-spacing .4em・緑）」と「`.cap` バッジ」という**小さな添え字ラベルの語彙**を持っています。この既存語彙を1つ増やすだけで、デザインを壊さずに正式名称を見出しタグ内へ入れられます。

### 解1（推奨）：h2 の中に、既存 `.sec-label` と対になる小さな漢字行を足す

**CSS（`</style>` 直前に追記）**

```css
/* 見出しの正式表記（sec-labelと対になる添え字。RPGの「サブタイトル」として自然） */
.h2-ja{
  display:block;
  font-size:11px;
  letter-spacing:.4em;
  color:var(--cyan);
  opacity:.8;
  margin-top:6px;
  font-weight:normal;
}
```

**HTML（452 / 491 / 515 / 531行）**

```html
<h2 class="pop">じぎょう ないよう<span class="h2-ja">事業内容 ／ LINE公式アカウントの構築・運用・開発</span></h2>
<h2 class="pop">やくいん しょうかい<span class="h2-ja">役員紹介</span></h2>
<h2 class="pop">かいしゃ がいよう<span class="h2-ja">会社概要 ／ 株式会社かみねんど</span></h2>
<h2 class="pop">おといあわせ<span class="h2-ja">お問い合わせ ／ ご相談・お見積り</span></h2>
```

- 見た目：ひらがなの大見出しはそのまま。下に緑ラベルと対称のシアン細字が1行入るだけ。上下でラベルに挟まれた構図になり、**むしろRPGのウィンドウらしさが増します**。
- 効果：`<h2>` タグの中に「事業内容」「会社概要」「役員紹介」「お問い合わせ」「LINE公式アカウントの構築・運用・開発」が入る。h2のキーワード価値がゼロから実用レベルへ。
- 注意：`aria-label` で漢字を補う案は検索評価に使われない前提が安全なので採りません。**可視テキストで入れる**のが確実です。

### 解2：`.sec-label` を英日併記にする（解1と併用可、CSS変更ゼロ）

```html
<p class="sec-label pop">QUEST LIST ／ 事業内容</p>
```

CSS を1行も触らずに漢字を出せますが、`<p>` なので見出しの重みは付きません。**解1が主、解2は補助**。

### 解3：h1 は社名のまま据え置く（積極的な判断）

`株式会社<br>かみねんど` に事業語が無いのは事実ですが、**トップのh1を社名純度100%に保つほうが指名検索（=いま最優先で取りに行く軸KW #1）には有利**です。事業語のh1は下層サービスページで取ります（1ページ1論点＝AIのファンアウト対策としても正解）。ただし `<br>` が「株式会社／かみねんど」に分断してしまう問題は残るので、**h1直下の可視テキスト（§4-4-(2)）で「株式会社かみねんどは、LINE公式アカウントの構築・運用・開発を行う会社です」と1文で言い切る**ことで補います。この1文は AI が定義文としてそのまま引用しやすい形でもあります。

### 解4：人名を見出しに昇格（491〜509行）

`<p class="name">大迫 龍太</p>` は h3 相当の意味を持っています。**CSSクラスはそのままにタグだけ h3 へ変更**すれば、見た目は1px も変わらずに人物エンティティが見出しになります。

```html
<h3 class="name">大迫 龍太</h3>
<h3 class="name">福井 創祈</h3>
```

`.name` のスタイル定義が `p.name` ではなく `.name` である前提です（実装時に該当CSSセレクタを確認して、必要なら `h3.name{font-size:…}` を1行足す）。

### 解5：下層ページのデザイン方針

- 共通CSS（現状 index.html にインライン9.7KB）を **`assets/rpg.css` に外部化**し、下層ページはこれを読む。ブランド一貫性を保ちつつ HTML を軽くできる（現状 privacy.html だけデザインが完全に別系統になっている問題も同時に解消）。
- **オープニング演出は下層ページには載せない**。全画面インタースティシャルはトップだけの「起動シーケンス」として意味があり、サービスページに付けるとCWVとUXの両方を壊します。
- 下層ページの見出しは、**h1 は漢字の実キーワード**（例：`LINE公式アカウントの構築・運用代行`）、h2 以下でひらがなラベル（`.sec-label`）とRPGウィンドウを使ってブランドを表現する。**世界観は装飾で出し、意味は見出しで出す**という役割分担にします。

---

## 6. コンテンツ拡充計画（優先度順）

> 「〜」「約」「幅」ではなく**実数**を書くことが、このサイトの最大の差別化になります（LINE側もマジシャン側も、上位競合の大半が幅でしか価格を書いていない）。ただし**数字は社長確認が取れたものだけ**（§9）。

### P0（受け皿の骨格・これが無いと何も始まらない）

| # | ページ | 狙う軸KW | 想定AI質問 | 必須要素 |
|---|---|---|---|---|
| 1 | `l-os.jp/`（製品LP） | L OS / LINE 運用 SaaS | Q3「L OSとは何ですか？」 | **冒頭に「L OS とは、〜です。」の定義文1文**。機能の見出しは名詞句。全てSSR/SSG |
| 2 | `l-os.jp/pricing` | L OS 料金 | Q1 | プラン表・実額・初期費用の有無 |
| 3 | `l-os.jp/compare/lstep` | **Lステップ 代替** | Q2 | L OS／Lステップ／エルメの3列表。移行手順。**最も競合が薄い顕在層** |
| 4 | `/line/`（index.html） | LINE公式アカウント 構築代行 | Q4 | h1漢字。4サービスの詳細（現状トップの①を独立・拡張）。無料診断CTA |
| 5 | `/line/price.html` | LINE公式アカウント 構築代行 費用 | Q1 | **実額**。競合が幅でしか書かない領域 |
| 6 | `/company/`（index.html） | 株式会社かみねんど | Q5 | 会社概要6項目＋沿革＋事業内容。**平文HTML**。既に事実密度が高いので最も低コストで最も効く |
| 7 | `/privacy-policy.html` | — | — | §4-5。フォームを持つ企業サイトとして必須 |

### P1（顕在層・競合が薄い区画）

| # | ページ | 狙う軸KW | 想定AI質問 | メモ |
|---|---|---|---|---|
| 8 | `/performer/exhibition.html` | 展示会 ブース 集客 マジシャン | Q6 | **競合ほぼ皆無・BtoB単価が高い。最推奨** |
| 9 | `/performer/`（index.html） | マジシャン 派遣 ＋地域 | Q7 | 演目・時間・人数・進め方 |
| 10 | `/performer/year-end-party.html` | 忘年会 余興 マジシャン | Q7 | **9〜11月集中。8月着手で間に合う季節ページ** |
| 11 | `/members/ryu.html` | 大迫龍太 マジシャン | Q9 | ProfilePage。`osako.kaminendo.art` と**相互リンク**してサブドメイン分散を緩和 |
| 12 | `/line/subsidy.html` | IT導入補助金 LINE公式アカウント | Q8 | 価格離脱層を拾う。**支援事業者登録の有無で書ける内容が変わる（要確認）** |
| 13 | `/faq.html` | — | Q1〜Q10 全部 | **可視HTMLのQ&A。FAQPageスキーマは入れない**（2026年5月にリッチリザルト終了）。設問文は§3-3をそのまま使う |

### P2（月2〜3本で積む・合算で効かせる）

| # | ページ群 | 狙うKW型 | メモ |
|---|---|---|---|
| 14 | `/line/[地域].html` | 地域×LINE | **拠点確定後**。名古屋・愛知・東海／または東京・葛飾・城東 |
| 15 | `/line/[業種].html` | 地域×業種×課題 | 美容室・整体/整骨院・学習塾・飲食・工務店。事例が言語化しやすい順 |
| 16 | `/case/*.html` | 導入事例 | **公開可否は要確認**。SEO・成約率・AI引用の全てに効く最も汎用性の高い資産 |
| 17 | `l-os.jp/blog/richmenu-*` | リッチメニュー 作り方/サイズ | 自社の確定描画エンジンと直結する数少ない大需要語 |
| 18 | `/performer/[シーン].html` | 新年会・歓送迎会・周年・二次会 | 競合ポータルは数十〜数百ページ持っている。ページ数差が流入差 |

**すべてのページ共通の実装ルール**
- h1・title は§3-3の**想定質問文に寄せる**（Ahrefs 140万プロンプト分析：引用URLのタイトル類似度 0.602 vs 非引用 0.484）
- 冒頭1〜2文で結論。「導入→背景→結論」の日本語作文順は使わない
- 1ページ1論点
- URLスラッグは自然言語英単語（`/line/price.html`。`?p=123` 型は不可。引用率 89.78% vs 81.11%）
- **可視HTMLに「最終更新日：2026年8月22日」を表示**（JSON-LD の `dateModified` だけでは不十分）
- 署名（大迫龍太／福井創祈）とプロフィールページへのリンク
- BreadcrumbList JSON-LD を各下層に

**下層ページ用 JSON-LD テンプレ（`/line/price.html` の例）**

```html
<script type="application/ld+json">
{
  "@context":"https://schema.org",
  "@graph":[
    {"@type":"BreadcrumbList","itemListElement":[
      {"@type":"ListItem","position":1,"name":"ホーム","item":"https://kaminendo.art/"},
      {"@type":"ListItem","position":2,"name":"LINE公式アカウントの構築・運用","item":"https://kaminendo.art/line/"},
      {"@type":"ListItem","position":3,"name":"料金"}
    ]},
    {"@type":"WebPage","@id":"https://kaminendo.art/line/price.html#webpage",
     "url":"https://kaminendo.art/line/price.html",
     "name":"LINE公式アカウントの構築代行はいくらかかりますか？｜株式会社かみねんど",
     "inLanguage":"ja",
     "datePublished":"2026-08-22","dateModified":"2026-08-22",
     "isPartOf":{"@id":"https://kaminendo.art/#website"},
     "about":{"@id":"https://kaminendo.art/#organization"},
     "author":{"@id":"https://kaminendo.art/#osako"}}
  ]
}
</script>
```

**`/members/ryu.html` は `ProfilePage`**（Google が現在サポート中の型）

```html
<script type="application/ld+json">
{
  "@context":"https://schema.org",
  "@type":"ProfilePage",
  "mainEntity":{"@id":"https://kaminendo.art/#osako"},
  "dateModified":"2026-08-22"
}
</script>
```

---

## 7. AIOに特化した施策

**大前提：AIO対策の本体は通常SEOです。** 日本国内実測でAI Overview引用の90.2%が検索10位以内から来ています。「AIO専用施策」に見えるものの大半は費用対効果が悪く、以下の5つに絞ります。

**① JS依存テキストの撲滅（技術・即効・最重要）**
Vercel×MERJ のサーバログ実測で、GPTBot / OAI-SearchBot / ClaudeBot / PerplexityBot はいずれもJSを実行しません（GPTBot は5億回超のフェッチで実行痕跡ゼロ）。現状 kaminendo.art の `#vision`（title の最重要コピー）と起動シーケンス文言、キャラのセリフはJS注入で、**AIからは存在しません**。§4-4-(1) で解消。`l-os.jp` の製品LPも必ず SSR/SSG で作ります。

**② 引用の原資を可視HTMLに置く**
AIは「幅」を引用しません。以下を**可視テキストとして**書きます（数値は§9で確定したものだけ）。
- 定義文：「**L OS とは、業界を問わないLINE公式アカウント運用システムです。**」の形。AIは定義文をそのまま引用します
- 実額の料金表、対応業種数、構築から公開までの日数、無料診断レポートのページ数（「10ページ超」は既にある数少ない具体値）
- 会社概要の6項目（法人番号 2011801047648 まで書いてあるのは強い。これは**そのまま維持**）
- **各ページに最終更新日を可視表示**

**③ 外部エンティティの確立（自社サイト外が本丸）**
AI検索でのブランド言及の大半は自社ドメイン外から来ます。日本語圏でAIが実際に引用しているドメインに載せるのが最短経路です。
| 施策 | 根拠 | 難度 |
|---|---|---|
| **ITreview / BOXIL に L OS を登録** | ITreviewの「Lステップ 競合・類似製品」に L OS が載っていないこと自体が機会損失。AIはこの手の一覧を高頻度で引用 | 低・無料枠あり |
| **PR TIMES で月1本**（設立、L OS 正式公開、機能追加など） | ChatGPT で約9.5万引用。日本語圏でプレスリリースは一次情報として扱われる | 中・有料 |
| **note で継続発信**（最低6ヶ月） | AI Overviews / ChatGPT / Copilot / Perplexity の4プラットフォームで上位 | 低・無料 |
| **LINEヤフー for Business のパートナー／正規代理店一覧** | 名古屋の総合アドが「東海No.1正規代理店」を訴求できているのはこの肩書のため。掲載条件を満たせるか要確認 | 要確認 |
| **「LINE運用代行おすすめN社」系メディアへの掲載打診** | AIが真っ先に参照するのはこの手の比較記事 | 中・広告枠 |

**④ `sameAs` で同一性を機械可読に**
「かみねんど」という語は nendo・紙粘土・pixiv絵師に強く占有されています。エンティティ解決を助けるため、**実在確認できたURLだけ**を `sameAs` に列挙します（§4-3-a）。特に法人番号公表サイト／gBizINFO の自社ページは、公的な同一性の裏付けとして価値が高い。

**⑤ やらないと決めるもの（判断の明文化）**
| 施策 | 判断 | 理由 |
|---|---|---|
| llms.txt | **作るが期待しない**（§4-7） | 137,000ドメイン実測で97%が0リクエスト。Google公式も非対応 |
| FAQPage スキーマ | **新規実装しない** | 2026年5月7日にリッチリザルト終了、8月にSearch Console APIからも削除。**可視HTMLのQ&Aセクション自体は有効なので `/faq.html` は作る** |
| WebSite + SearchAction | **入れない** | 2024年11月廃止 |
| AI検索専用の別ページ生成 | **やらない** | 根拠なし＋クローキング判定リスク |
| 「AI向け特別スキーマ」を謳う有料ツール | **買わない** | Google公式が「特別なスキーマは不要」と明言。Ahrefsの対照実験（1,885 vs 約4,000ページ、DiD）でリフトはゼロ |

---

## 8. 実装ロードマップ

### Phase 1：即日（合計 3〜4時間・Claude Code が直接編集）

| # | 作業 | 工数 | 期待効果 |
|---|---|---|---|
| 1-1 | **既存の未コミット変更をコミット＆push**（title/desc/canonical/OGP/JSON-LD/robots.txt/sitemap.xml） | 5分 | **最大。作業済みのものが本番に出ていない状態を解消** |
| 1-2 | `l-os.jp` に `app/robots.ts` ＋ `/t/`・`/admin` の noindex | 30分 | 管理画面インデックスの事故を防止（リスク側の最優先） |
| 1-3 | `404.html` 新規（§4-1） | 20分 | 英語GitHubページの解消 |
| 1-4 | `#vision` 静的化＋タイプライタ1行修正、hero-copy に事業語（§4-4-1,2） | 20分 | **AIクローラに最重要コピーが見えるようになる** |
| 1-5 | h2 に漢字併記（§5解1）＋人名を h3 へ（§5解4） | 30分 | h2のキーワード価値が0→実用レベル |
| 1-6 | フッターに `l-os.jp`／privacy リンク（§4-4-4）、会社概要に3行追加（§4-4-3） | 15分 | 孤立ページ解消・自社SaaSとの紐付け |
| 1-7 | ロゴ width/height、canvas の aria、`prefers-reduced-motion`、オープニングのキーボード解除（§4-4-5〜8） | 40分 | CLS・アクセシビリティ |
| 1-8 | **GSC（ドメインプロパティ）＋ Bing WMT 登録、サイトマップ送信** | 40分 | **計測ゼロの解消。これ以降すべての判断の根拠になる** |
| 1-9 | GitHub Pages の Custom domain 設定で **www の証明書を発行**（または www の DNS を削除） | 20分 | 到達不能な入口の解消 |

### Phase 2：数日〜2週間（合計 20〜30時間）

| # | 作業 | 工数 | 期待効果 |
|---|---|---|---|
| 2-1 | `privacy-policy.html` 新規＋既存 privacy.html に責任主体明記（§4-5） | 2h | 企業サイトとしての体裁。信頼性 |
| 2-2 | 共通CSSを `assets/rpg.css` に外部化 | 2h | 下層ページ量産の前提。HTML軽量化 |
| 2-3 | `/company/` `/line/` `/line/price.html` `/members/ryu.html` の4本（§6 P0） | 10h | **受け皿の骨格。指名検索と費用クエリの着地点** |
| 2-4 | `l-os.jp/` 製品LP＋`/pricing`＋`/compare/lstep`（SSG） | 8h | **最大の武器を検索上に出す。「Lステップ代替」は最も競合が薄い** |
| 2-5 | sitemap.xml 更新＋各ページに BreadcrumbList / ProfilePage JSON-LD | 2h | クロール効率 |
| 2-6 | GA4 導入＋参照元にAI検索ドメインを分離 | 1h | 効果測定 |
| 2-7 | ITreview / BOXIL に L OS 登録 | 2h | **AI引用への最短経路** |

### Phase 3：継続（月あたり 8〜12時間）

| # | 作業 | ペース | 期待効果 |
|---|---|---|---|
| 3-1 | `/performer/exhibition.html` → `/performer/year-end-party.html` → `/faq.html` → `/line/subsidy.html` | 月2〜3本 | 競合が薄い区画から順に |
| 3-2 | 地域×業種ページを積む（拠点確定後） | 月2〜3本 | 20〜30本で合算流入 |
| 3-3 | **PR TIMES で月1本** | 月1 | ChatGPTが一次情報として扱う |
| 3-4 | **note で継続発信**（最低6ヶ月） | 月2〜4本 | 4プラットフォームで上位 |
| 3-5 | 導入事例を追加（公開許諾が取れ次第） | 随時 | SEO・成約率・AI引用の全てに効く |
| 3-6 | 既存ページの実質更新＋更新日の書き換え | 四半期ごと | 鮮度 |
| 3-7 | 月次でAI検索に想定質問10本を投げて記録（§10） | 月1・30分 | 効果測定 |

**期待できる時間軸（正直に）**：指名検索（`株式会社かみねんど`）はPhase 1〜2で数週間以内に取れる見込み。`Lステップ 代替` 系は Phase 2 完了から2〜4ヶ月。地域×業種のロングテールは Phase 3 を6ヶ月続けて合算で効き始める。**ヘッド語は狙わないので、いつまでも1位になりません。それでよい、というのが方針です。**

---

## 9. 社長に確認が必要な事項

**まず1問だけ答えてほしいもの → Q1。** これが決まらないと地域ページ・GBP・NAP統一・`sameAs` の全部が動かせません。

### 事実確認（Claude Code では調べられないもの）

| # | 確認事項 | なぜ必要か |
|---|---|---|
| **Q1** | **拠点は東京都葛飾区か、愛知県名古屋近郊か。両方なら本店／営業所のどちらがどちらか** | サイト記載は葛飾区、今回の依頼文には「名古屋近郊」とあり矛盾。地域KW・GBP・NAP・`address` 構造化が180度変わる |
| Q2 | 公開してよい電話番号はあるか（無いなら「無い」で確定させる） | `telephone` を入れるか、会社概要に「メールのみ」と書くか |
| Q3 | **料金の実額を公開してよいか。よい場合、LINE構築／運用／開発それぞれいくらか** | 競合が全社「幅」でしか書いていない唯一の差別化軸。`/line/price.html` の成否そのもの |
| Q4 | 公開してよい実績・事例（社名／業種のみ／数字だけ、いずれの粒度でも） | 事実の創作は厳禁のため、無ければ事例ページは作らない |
| Q5 | L OS の**構築実績件数・対応業種数・導入から公開までの日数** | AIが引用する具体値。無ければ書かない |
| Q6 | `privacy.html` は App Store に登録済みのURLか | 該当するなら現URLを絶対に動かせない（審査リスク） |
| Q7 | 公式SNS（X / Instagram / YouTube / note / Facebook）は存在するか | `sameAs` に入れられるのは実在するものだけ。無いなら「作るかどうか」の判断（Q13へ） |
| Q8 | LINEヤフー for Business の正規代理店／IT導入支援事業者の登録はあるか | 名古屋の競合が「東海No.1正規代理店」で優位に立っている肩書。補助金ページの内容も変わる |
| Q9 | 顔写真を掲載してよいか（現状ドット絵canvasのみで実写ゼロ） | E-E-A-T。役員ページの説得力 |
| Q10 | お問い合わせ内容の保存期間 | プライバシーポリシー第5条に必要 |
| Q11 | 公式LINEの友だち情報をどこまで取得・利用しているか | 同第7条に必要 |

### 経営判断が必要なもの

| # | 判断事項 | 選択肢と影響 |
|---|---|---|
| Q12 | **`l-os.jp` のルートを製品LPにしてよいか** | 現在ログイン画面のみ。LPに変えるなら既存の運用者導線を `/login` に移す必要があり、テナントへの周知が要る。**やらないと最大の武器が検索上に存在しないまま** |
| Q13 | PR TIMES（月1本）・note の運用に月いくら／何時間かけられるか | AI引用への効果が最も大きい外部施策だが、継続が前提。単発では効かない |
| Q14 | 「魔法の公式LINE」を主表記のまま残すか | 独自造語は指名検索に強い一方、一般名詞「LINE公式アカウント」の検索需要を拾えない。**推奨は併記**（title/descriptionは既に一般名詞に寄せ済み） |
| Q15 | `osako.kaminendo.art` / `fukui.kaminendo.art` を将来 `kaminendo.art/members/` に統合するか | サブドメインは別サイト扱いでドメイン評価が分散する。当面は相互リンクで緩和し、統合は後日判断 |
| Q16 | `www.kaminendo.art` を生かすか殺すか | 現在HTTPSで証明書エラー＝到達不能。**中途半端に生きているのが最悪**。生かす（証明書発行）／殺す（DNS削除）のどちらかに倒す |
| Q17 | 商品開発・経営コンサルの2事業はSEOの主戦場にしない、でよいか | ヘッド語は大手・士業が寡占。①②の実績記事内で「自分で作って売っている／自分で経営している」証明素材として機能させる方が投資効率が高い |
| Q18 | GA4 / GSC / Bing はどのGoogleアカウントで作るか（`kaminendo.biz@gmail.com` か `osako@kaminendo.art` か） | 後から移管が面倒なので最初に決める |

---

## 10. 効果測定の方法

### 10-1. 登録するもの（Phase 1-8 で実施）

| ツール | 設定 | 見るもの |
|---|---|---|
| **Google Search Console** | **ドメインプロパティ** で `kaminendo.art` を登録（DNSのTXTレコード。`osako.` `fukui.` も自動で含まれる）。別途 `l-os.jp` も | ①対象KWの平均掲載順位 ②インデックス済みページ数 ③クロール統計 |
| **Bing Webmaster Tools** | GSCからインポート可。サイトマップ送信＋**IndexNow** キーファイルをリポジトリ直下に設置 | Copilot / DuckDuckGo / Yahoo の基盤。設定30分・反映は数十分 |
| **GA4** | `index.html` と下層ページ、`l-os.jp` に設置 | 流入元・行動・コンバージョン（フォーム送信・LINE遷移） |

### 10-2. 毎月見る4つの数字

**① 対象キーワードの掲載順位（GSC／最重要）**
AI Overview引用の90.2%が10位以内から来るため、**順位がAIO可視性の最良の先行指標**です。§3-1の軸KW 10本を GSC の「検索パフォーマンス > クエリ」で追跡し、10位以内に入った本数を月次で記録します。

**② インデックス済みページ数（GSC「ページ」）**
Phase 2〜3の進捗そのもの。2 → 10 → 30 と増えているか。`/t/` `/admin` が誤ってインデックスされていないかも同時に確認。

**③ AI検索からの参照流入（GA4）**
GA4 の探索で参照元に以下を含む行を分離。ただし**AIアプリ内リンクはリファラを落とすことがあり必ず過小計上になる**（Cloudflareも同じ制約を指摘）ので、絶対値ではなく増減の傾向だけを見ます。
`chatgpt.com` / `perplexity.ai` / `copilot.microsoft.com` / `gemini.google.com` / `claude.ai`

**④ 実際にAIに聞いて記録する（最も確実・ツール不要）**
月1回30分、§3-3の想定質問Q1〜Q10を **ChatGPT / Perplexity / Google AI Mode** に日本語でそのまま投げ、以下を表に記録します。

| 日付 | 質問 | ChatGPT | Perplexity | AI Mode | 引用元URL |
|---|---|---|---|---|---|
| 2026-09-01 | LINE公式アカウントの構築代行はいくらかかりますか？ | 自社なし | 自社なし | 自社なし | （競合3社を記録） |

「自社が挙がったか」「挙がらなかった場合どこが引用されているか」の2列が、次に作るべきコンテンツを直接教えてくれます。

### 10-3. AIクローラのアクセス確認（GitHub Pages の制約と代替手段）

**GitHub Pages ではサーバログが取得できません**（robots.txt が許可でもWAF等が403を返している事故が業界最頻ですが、それを直接確認する手段がありません）。代替として：
- **GSC のクロール統計**（設定 > クロールの統計情報）で Googlebot の応答コードを確認
- **Bing WMT のクロール情報**で Bingbot を確認
- OpenAI/Anthropic/Perplexity については、外部から User-Agent を偽装して 200 が返るかを月1回だけ手動確認（`curl -A "OAI-SearchBot" -I https://kaminendo.art/`）。**これはbotが実際に来ているかの証明にはならず、「拒否設定になっていないこと」の確認にとどまります**
- `l-os.jp` は Vercel なので、こちらは Vercel のログでAIクローラの実アクセスを確認できます

### 10-4. 判定基準（3ヶ月後・6ヶ月後）

| 時点 | 合格ライン |
|---|---|
| 1ヶ月後 | GSCでインデックス済み ≥ 8ページ／`株式会社かみねんど` で1位／`l-os.jp` の `/t/` `/admin` が非インデックス |
| 3ヶ月後 | 軸KW 10本のうち **3本が20位以内**／AI質問チェックで **1問以上** 自社が挙がる／ITreview に L OS 掲載済み |
| 6ヶ月後 | 軸KW 10本のうち **3本が10位以内**（＝AIO引用の前提を満たす）／AI質問チェックで **3問以上**／インデックス済み ≥ 25ページ |

---

**使用Skill：なし**（本タスクは既存4調査の統合と実装計画の設計であり、該当するSkillの適用範囲外。実際に行ったのは対象リポジトリの実測確認のみ）

**確認した実ファイル（すべて絶対パス）**
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\index.html`（45,456 bytes・未コミット変更あり）
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\robots.txt`（未追跡・新規）
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\sitemap.xml`（未追跡・新規）
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\privacy.html`
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\assets\ogp.png`（1200×630・未追跡）
- `C:\RYUフォルダ\021_HP制作\KAMINENDO\assets\logo_white.png`