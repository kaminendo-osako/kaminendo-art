# 計測の初期設定（Search Console / GA4 / Bing）

使うGoogleアカウント: **kaminendo.biz@gmail.com**（社長判断で確定）
対象サイト: `https://kaminendo.art/` と `https://l-os.jp/` の**2つとも**登録します。

> 現状は計測がゼロです。これを入れないと「SEO対策の効果があったのか」を判定する手段がありません。
> 所要 30〜40分。1回やれば以後は見るだけです。

---

## 1. Google Search Console（最優先・これだけでもやる）

検索での見え方（表示回数・クリック・順位・インデックス状況）が分かります。

### kaminendo.art

1. https://search.google.com/search-console に kaminendo.biz@gmail.com でログイン
2. 「プロパティを追加」→ **URLプレフィックス** を選び `https://kaminendo.art/` を入力
3. 所有権の確認方法で **HTMLタグ** を選ぶ → `<meta name="google-site-verification" content="…">` が表示される
4. その1行を `index.html` の **`▼ここに貼る` のコメント内**（`<meta name="theme-color">` の直後）に貼り、コメントの外に出す
5. `git add -A && git commit -m "GSC所有権確認" && git push` → 反映を待って「確認」を押す
6. 確認できたら、左メニュー「サイトマップ」→ `sitemap.xml` と入力して送信

### l-os.jp

1. 同様に `https://l-os.jp/` をURLプレフィックスで追加
2. Next.js なので、HTMLタグ方式なら `line-os/app/layout.tsx` の `metadata` に追記します:
   ```ts
   verification: { google: "ここにGSCが出す値" },
   ```
3. `git push`（Vercelが自動デプロイ）→「確認」を押す
4. サイトマップに `sitemap.xml` を送信

> DNS方式でもOKです。Cloudflareで両ドメインを管理しているなら、DNSにTXTを1本足すだけで、
> サブドメインを含めてまとめて確認できるのでむしろ楽です。

---

## 2. Google アナリティクス（GA4）

誰がどこから来て、何を見たかが分かります。

1. https://analytics.google.com にログイン →「管理」→「作成」→「プロパティ」
2. プロパティ名: `かみねんど`（レポートのタイムゾーン: 日本、通貨: 円）
3. データストリーム →「ウェブ」→ `https://kaminendo.art` を登録 → **測定ID `G-XXXXXXX`** が出る
4. その測定IDで、`index.html` のコメント内にあるGA4タグを貼り替えて有効化
5. l-os.jp は別のデータストリームを作り、`line-os/app/layout.tsx` に `<Script>` で入れる
   （Next.js は `next/script` を使う。入れる場合は指示ください、こちらで実装します）

---

## 3. Bing Webmaster Tools（任意・5分）

ChatGPT検索の一部はBingの索引を使うため、AI検索の観点で入れておく価値があります。

1. https://www.bing.com/webmasters にログイン
2. 「Search Consoleからインポート」を選ぶと、上の設定がそのまま取り込めます

---

## 毎月見る数字（4つだけ）

| 数字 | どこで見る | 何が分かる |
|---|---|---|
| 表示回数・クリック数 | Search Console → 検索パフォーマンス | 検索でどれだけ見られているか |
| 「株式会社かみねんど」の順位 | 同上 → クエリで絞る | 指名検索を取り切れているか |
| インデックス済みページ数 | Search Console → ページ | 作ったページが検索に載ったか |
| 公式LINEへの遷移数 | GA4 → イベント | 問い合わせにつながっているか |

### 判定の目安

- **1ヶ月後**: kaminendo.art と l-os.jp の両方がインデックスされ、「株式会社かみねんど」で1位
- **3ヶ月後**: 「L OS LINE」など製品名まわりで表示が出はじめる
- **6ヶ月後**: ロングテール（地域×業種）からの流入が発生
