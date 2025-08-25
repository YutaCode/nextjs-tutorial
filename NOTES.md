# 学習メモ: Next.js Dashboard Tutorial

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