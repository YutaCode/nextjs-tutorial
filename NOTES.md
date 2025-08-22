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