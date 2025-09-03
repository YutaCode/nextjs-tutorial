# 学習メモ: Next.js Dashboard Tutorial

## React標準とNext.js拡張の違い

- **Reactが提供する標準フック**
  - `useState`, `useEffect`, `useContext`, `useRef` など
  - UIや状態管理の基本機能に関わるもの

- **Next.jsが提供する拡張フック（`next/navigation`など）**
  - `useRouter` → ページ遷移・URL更新を行う
  - `usePathname` → 現在のパスを取得
  - `useSearchParams` → URLの検索パラメータを取得/監視
  - App Router（`app/` ディレクトリ構造）で利用する

👉 React = UIライブラリの基礎  
👉 Next.js = Reactを拡張してルーティングやSSRなどWebアプリ向け機能を追加したフレームワーク

## Chapter 1: Getting Started
- `create-next-app`でスターターを作成
- `pnpm dev` でローカル起動できることを確認

## Chapter 2: CSS Styling
- `global.css` = 全体に効くスタイル
- `Tailwind` = クラスベースで即スタイル適用
- `CSS Modules` = コンポーネント単位でスコープ化される
- `clsx` = 条件付きでクラスを切り替えるのに便利

## Chapter 3: Fonts & Images
- `next/font` = フォントを最適化してレイアウトシフト防止
- `next/image` = 画像を最適化してレスポンシブやlazy load対応

## Chapter 4: Layouts

- Create the `dashboard` routes using file-system routing  
  → フォルダ構造がそのまま URL になる。`/app/dashboard/page.tsx` → `/dashboard`

- Understand the role of folders and files when creating new route segments  
  → `page.tsx` は必須、`layout.tsx` は任意で共通UIを定義できる

- Create a nested layout that can be shared between multiple dashboard pages  
  → `/dashboard/layout.tsx` を作成すると、配下のページに共通のサイドナビなどを配置できる

- Understand what colocation, partial rendering, and the root layout are  
  → colocation = 関連ファイルを同じ場所にまとめられる  
  → partial rendering = ページ遷移時に layout は再描画されず、page だけ更新  
  → root layout = アプリ全体に共通する枠（必須）

## Chapter 5: Navigation
- `<Link>`: リロードなしでページ遷移できる
- `usePathname()`: 今のURLを取得（アクティブリンクに使う）
- `clsx`: 条件付きでクラスを当てられる
- ナビゲーションは部分レンダリング＋プリフェッチで高速

## Chapter 6: Database Setup
- Vercel に GitHub リポジトリを接続（自動デプロイ有効化）
- Storage → Postgres(Neon) で DB 作成（リージョンは近場：例 Singapore）
- `.env` に `POSTGRES_URL` などの接続情報を追加（Show secret → コピー）
- `/seed` にアクセスして初期データ投入（成功後、seed 用ファイルは削除）
- `/query` の API ルートで DB 接続を確認（OKならこの確認用ルートも削除可）
- メモ：`route.ts` は API エンドポイントを作る場所（DB クエリはここから実行）

## Chapter 7: Fetching Data
- データ取得方法の種類  
  → API レイヤー（`route.ts` で作成、クライアントから安全にアクセス）  
  → Server Components（`async/await` で直接 DB にアクセス可能、秘密保持◎）  
  → SQL 直書き（`postgres.js` を使ってクエリを投げる）

- サーバーコンポーネントの利点  
  → DB 接続情報をクライアントに渡さないで済む  
  → `useEffect` や `useState` を使わずシンプルに書ける  

- 非同期処理  
  → `await`: データが返るまで処理を待つ  
  → `Promise.all()`: 複数のリクエストを並列実行してウォーターフォールを回避  

- 実装例  
  → `RevenueChart`: 月ごとの売上を DB から取得してグラフ表示  
  → `Card`: 支払済み / 保留中の請求書合計、総請求数、顧客数を SQL で集計して表示  

## Chapter 8: Static and Dynamic Rendering
- Static Rendering = ビルド時 or 再検証時に生成（キャッシュを返す）
  - 高速 / サーバー負荷軽減 / SEO に強い
  - デメリット: 再検証されるまで最新データが反映されない
  - 適用例: ブログ記事、商品ページ（共通データ）

- Dynamic Rendering = リクエストごとにサーバーで生成
  - リアルタイム性 / ユーザーごとの個別データに強い
  - Cookie, URL パラメータなどリクエスト時の情報を利用できる
  - デメリット: 遅いフェッチに全体が引っ張られる
  - 適用例: ダッシュボード、頻繁に変化するデータ

- 実験: fetchRevenue() に3秒delayを入れると全体が遅延
  → Dynamic は最も遅いフェッチに依存する

## Chapter 9: Streaming, Suspense, Skeletons, Route Groups

- **Streaming**
  - ページを小さなチャンクに分けて、準備できた部分から順に表示
  - 遅いフェッチが全体をブロックしない

- **loading.tsx**
  - ページロード中に表示する代替 UI を定義する特殊ファイル
  - 内部的には React Suspense が利用されている
  - 静的な部分はすぐ表示され、動的な部分は後から差し込まれる

- **Skeleton UI**
  - 開発者が用意する「仮の UI」
  - Next.js が自動生成するわけではない
  - 本物のUIに似せることで UX を自然に見せられる

- **Route Groups**
  - URL に影響しない整理フォルダ（`()で囲む`）
  - 例: `/dashboard/(overview)/page.tsx` → URLは `/dashboard`
  - `loading.tsx` の適用範囲を限定できる
  - 大規模開発ではセクションやチーム単位での整理にも使える

- **Suspense**
  - コンポーネント単位でデータフェッチを分離してラップできる
  - `fallback` に Skeleton を渡すと遅延中だけ仮UIを表示
  - ページ全体をラップ → 遅延が1つでもあると全体が遅れる  
  - 全コンポーネントをラップ → UI がバラバラに「ポンッ」と出現し不自然  
  - セクションごとにラップ → 自然な段階的ロード（推奨）

  ## Chapter 10: Partial Prerendering (PPR) & Data Fetching Summary

- **Streaming の課題**
  - ページ内に動的処理が1つでもあるとルート全体が Dynamic 扱いになる
  - 「先に出すUI」も Dynamic の一部になるため、遅延に引っ張られる可能性がある

- **Partial Prerendering (PPR)**
  - Next.js 14 で導入された実験的機能（`next@canary`）
  - `next.config.ts` に `experimental.ppr = 'incremental'` を追加
  - ルート単位で `export const experimental_ppr = true` を設定
  - **静的シェル**を即座に返し、**動的部分**は Suspense を通してあとからストリーミング
  - Suspense = 静的と動的の境界線
  - コード変更は不要、Suspense でラップしていれば Next.js が自動で仕分け

- **メリット**
  - 静的の速さ + 動的の柔軟さを両取り
  - 本番環境でパフォーマンス改善が期待できる
  - 将来的にデフォルトのレンダリングモデルになる可能性あり

- **Data Fetching Optimization Summary**
  1. サーバーと同じリージョンにDBを作成し、レイテンシ削減
  2. React Server Components でサーバー側フェッチ（秘密保持 & JSバンドル削減）
  3. SQL で必要なデータだけを取得（転送量 & JS処理削減）
  4. JavaScript でのデータフェッチを並列化
  5. Streaming で遅延が全体をブロックするのを防止
  6. データフェッチを必要なコンポーネント単位に移動し、静的/動的を分離

## Chapter 11: Search & Pagination

### 🔍 検索機能
- **やりたいこと**  
  入力ボックスに文字を入れると、テーブルがその文字に合うデータだけ表示されるようにする。  

- **どうやって実現？**  
  1. 入力が変わるたびに **`handleSearch`** 関数が動く  
  2. 文字を **URL の `query` パラメータ** に入れる（例: `?query=lee`）  
  3. サーバーは URL の `query` を読んで、必要なデータだけ返す  
  4. テーブルは新しいデータを受け取って更新される  

👉 **ポイント**: URL に状態を保存しているから、ページをリロードしたり共有しても検索結果が維持される。  

---

### 📄 ページネーション
- **やりたいこと**  
  請求書がいっぱいあると全部は表示できないので、1ページ6件ずつに区切ってページを切り替えられるようにする。  

- **どうやって実現？**  
  1. URL に **`page` パラメータ** を追加（例: `?query=lee&page=2`）  
  2. サーバーは `page` と `query` を両方見て、対象のデータを返す  
  3. `<Pagination>` コンポーネントが次ページ・前ページのリンクを作る  
  4. ページ切り替えを押すと URL が更新 → サーバーから新しいデータを取得  

👉 **ポイント**: ページ番号も URL で管理しているから、状態がわかりやすい。  

---

### ⏱ Debouncing（デバウンス）
- **問題点**  
  入力のたびにサーバーにリクエストすると「l」→「le」→「lee」みたいに毎回送信されて効率が悪い。  

- **解決方法**  
  「入力が止まってから 300ms 待って」サーバーにリクエストするようにする。  
  → 無駄なリクエストを減らせる。  

---

### 💡 まとめ
- URL に検索ワードやページ番号を保存することで **共有・リロードしても状態が残る**  
- 検索とページネーションを **サーバーサイドで安全に処理** できる  
- `useRouter` でクライアント遷移を実現して、体験をなめらかに  
- Debouncing でリクエスト数を減らして効率化  

## Chapter 12: Server Actions

### Server Actionsとは
- サーバー上で直接非同期処理を実行できる仕組み  
- `<form action={fn}>` でサーバー関数を呼べる（APIルートを作らなくてOK）  
- React/Next.js が暗号化や入力チェックを自動的にサポートし、セキュリティ面を強化  
- 読み取りは **Server Components**、書き込みは **Server Actions** と役割を分けて使う  

---

### フォーム送信とFormData
- `<form>` 送信時に **`FormData` オブジェクト** がサーバー関数に渡る  
- `formData.get('fieldName')` で値を取り出せる  
- 出力はブラウザではなく **Next.js サーバーのターミナル** に表示される  

---

### バリデーション（Zodの活用）
- `zod` ライブラリでフォームのスキーマを定義  
- 送信された `FormData` を `schema.parse()` で型チェック＆変換  
- 例:  
  - 文字列を数値に変換 → `z.coerce.number()`  
  - ステータスを列挙型で制限 → `z.enum(['pending', 'paid'])`  

---

### DB書き込みの流れ
1. `formData` を受け取る  
2. Zod でバリデーション  
3. 金額を整数化（例: ドル→セントに換算して保存）  
4. 日付を生成（`new Date().toISOString().split('T')[0]` で `YYYY-MM-DD` に整形）  
5. SQL (`postgres.js`) で `INSERT` / `UPDATE` を実行  

---

### キャッシュの再検証とリダイレクト
- データ変更後はキャッシュをクリアして最新の一覧を表示する必要がある  
- `revalidatePath('/dashboard/invoices')` でキャッシュを無効化  
- `redirect('/dashboard/invoices')` でユーザーを一覧に戻す  

---

### Update の実装ポイント
- 動的ルート `[id]/edit/page.tsx` を作成し、`params.id` を受け取る  
- 既存データをDBから取得してフォームの `defaultValue` にセット  
- `updateInvoice.bind(null, id)` で「id を固定した関数」を作り `<form action>` に渡す  
- hidden input でも可能だが、セキュリティ的に `bind` が推奨  

---

### Delete の実装ポイント
- `DeleteInvoice` コンポーネントを `buttons.tsx` に実装  
- `deleteInvoice.bind(null, id)` を `<form action>` に渡して削除処理を実行  
- 削除後は `revalidatePath` で最新状態に更新  

---

### まとめ
- Server Actions = **データ変更系処理 (CUD)** を安全に実装する仕組み  
- APIルートを作らなくてもフォームから直接サーバー処理を呼べる  
- `bind` を使って安全にIDを渡すのが重要  
- Create / Update / Delete の一連の流れを通して、フルスタックなデータ操作が可能になった  