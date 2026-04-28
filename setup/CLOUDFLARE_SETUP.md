# Cloudflare Workers プロジェクト初期セットアップ手順

このドキュメントは、Cloudflare Workers にアプリを Wrangler + GitHub Actions で
自動デプロイする環境を構築するための手順です。

Claude Code は、この手順に従って初期セットアップを行ってください。
各ステップでユーザーに確認を取りながら進めてください。

---

## 前提条件の確認

まずユーザーに以下を確認してください。

1. Cloudflare アカウントは作成済みか?
2. このプロジェクトの用途は何か?(API サーバー、Webアプリ、静的サイトなど)
3. データベースは必要か?(必要なら D1 を使う)
4. Wrangler を初めて使うか?

---

## 推奨フレームワーク

シンプルな Webアプリ・API サーバーの場合、**Hono** フレームワークを推奨する。

- 軽量で Cloudflare Workers との相性が良い
- Express.js に似た書き方ができる
- TypeScript 対応

---

## ステップ1: Wrangler のインストールとログイン

ユーザーに以下のコマンドを案内する。

```bash
npm install -g wrangler
wrangler login
```

`wrangler login` はブラウザが開いて Cloudflare 認証を求めるため、ユーザー側で実施する必要がある。

---

## ステップ2: プロジェクトの初期化

リポジトリのルートで以下を実行するようユーザーに案内する。

```bash
npm create cloudflare@latest -- my-app
```

対話形式で以下を選択してもらう。

- **What would you like to start with?**: `Hello World example`(シンプル)または `Framework Starter`(Hono など)
- **Which language do you want to use?**: TypeScript 推奨(JavaScript でもOK)
- **Do you want to deploy your application?**: **No**(まだデプロイしない)

これで以下の構造が作成される。

```
.
├── src/
│   └── index.ts            ← メインのコード
├── wrangler.toml           ← Workers の設定
├── package.json
├── tsconfig.json
└── node_modules/
```

---

## ステップ3: wrangler.toml の確認

`wrangler.toml` の中身を確認し、必要に応じて以下を修正する。

```toml
name = "my-app"                    ← Workers の名前(URL の一部になる)
main = "src/index.ts"
compatibility_date = "2025-01-01"  ← 適切な日付に
```

---

## ステップ4: ローカルでの動作確認

```bash
npm run dev
```

ブラウザで `http://localhost:8787` を開いて、レスポンスが返るか確認する。

---

## ステップ5: 手動デプロイ(初回確認用)

```bash
npm run deploy
```

成功すると `https://my-app.〇〇〇.workers.dev` のような URL が払い出される。
実際にアクセスして動作確認する。

---

## ステップ6: GitHub Actions の設定

`.github/workflows/deploy.yml` に以下を作成する。

```yaml
name: Deploy to Cloudflare Workers

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Deploy to Cloudflare Workers
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

---

## ステップ7: GitHub Secrets の登録

### Cloudflare API トークンの取得

1. Cloudflare ダッシュボードを開く
2. 右上のアイコン → **My Profile** → **API Tokens**
3. **Create Token** をクリック
4. **Edit Cloudflare Workers** テンプレートを選択
5. **Continue to summary** → **Create Token**
6. 表示されたトークンをコピー(この画面を閉じると二度と見られないので注意)

### Cloudflare Account ID の取得

1. Cloudflare ダッシュボードのトップページ
2. 右側のサイドバーに表示されている **Account ID** をコピー

### GitHub Secrets への登録

GitHub のリポジトリページで以下を行う。

1. **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret** で以下を 2 つ登録
   - Name: `CLOUDFLARE_API_TOKEN`、Secret: 上で取得した API トークン
   - Name: `CLOUDFLARE_ACCOUNT_ID`、Secret: 上で取得した Account ID

---

## ステップ8: 動作確認

1. 何か小さな変更をして main ブランチにコミット&プッシュ
2. GitHub の Actions タブを開いて、ワークフローが成功するか確認
3. `https://my-app.〇〇〇.workers.dev` にアクセスして変更が反映されているか確認

---

## D1 データベースを使う場合(オプション)

データベースが必要な場合は、Cloudflare D1(SQLite ベース)を使う。

### D1 の作成

```bash
wrangler d1 create my-database
```

実行結果に表示される `database_id` を `wrangler.toml` に追記する。

```toml
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "ここに表示された ID を貼る"
```

### マイグレーションの実行

```bash
wrangler d1 execute my-database --file=./schema.sql
```

---

## トラブル対応

### `wrangler login` が失敗する場合
- ブラウザが自動で開かない場合、ターミナルに表示される URL を手動でブラウザに貼って認証する。

### デプロイ時に「Account ID is required」エラー
- `wrangler.toml` に `account_id` を追記するか、環境変数 `CLOUDFLARE_ACCOUNT_ID` を設定する。

### GitHub Actions で `Authentication error` が出る場合
- Secrets の `CLOUDFLARE_API_TOKEN` の値が正しいか確認。
- API トークンの権限が「Edit Cloudflare Workers」になっているか確認。

### 独自ドメインを使いたい場合
- Cloudflare ダッシュボード → Workers & Pages → 該当 Worker → Settings → Triggers → Custom Domains から追加。

---

## Render との違い(参考)

| 項目 | Render | Cloudflare Workers |
|------|--------|-------------------|
| 無料プラン | あり(スリープ有) | あり(スリープなし) |
| 起動速度 | スリープから30秒〜1分 | 常時即応答 |
| 言語 | Python / Node / Go など | JavaScript / TypeScript |
| データベース | 別サービス連携 | D1(内蔵) |
| 自動アクセス問題 | あり(規約違反になる) | スリープしないので不要 |

Render から移行する場合、Python アプリは TypeScript / JavaScript で書き直す必要がある点に注意。

---

## 完了後の状態

- main ブランチに push するだけで Cloudflare Workers に自動デプロイされる
- 無料プランでもスリープしないので、常に即応答
- 自動アクセスの設定は不要(過去のトラブル再発防止)
