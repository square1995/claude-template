# Node.js プロジェクト初期セットアップ手順

このドキュメントは、シンプルな Node.js プロジェクト(ローカル開発用ツール、
スクリプト、CLIなど)の初期セットアップ手順です。

特定のホスティング先を想定しないシンプルな構成。
Webアプリとしてホスティングしたい場合は、用途に応じて以下を参照する。

- Cloudflare Workers にデプロイ → `CLOUDFLARE_SETUP.md`
- 静的サイトとして公開 → `FIREBASE_SETUP.md`

Claude Code は、この手順に従って初期セットアップを行ってください。
各ステップでユーザーに確認を取りながら進めてください。

---

## 前提条件の確認

まずユーザーに以下を確認してください。

1. このプロジェクトの用途は何か?(スクリプト、CLI、ローカルツールなど)
2. Node.js は既にインストール済みか?
3. TypeScript を使うか、JavaScript のままで良いか?

---

## ステップ1: Node.js のインストール確認

ユーザーに以下のコマンドで確認してもらう。

```bash
node --version
npm --version
```

バージョンが表示されれば OK。Node.js 20 以上を推奨。

インストールされていない場合は、[https://nodejs.org/](https://nodejs.org/) からインストールしてもらう。

---

## ステップ2: package.json の作成

リポジトリのルートで以下を実行する。

```bash
npm init -y
```

`package.json` が自動作成される。必要に応じて以下のフィールドを編集する。

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "プロジェクトの説明",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "node --watch src/index.js"
  },
  "license": "MIT"
}
```

---

## ステップ3: ディレクトリ構成の作成

```
.
├── src/
│   └── index.js          ← メインのコード
├── package.json
├── .gitignore
└── README.md
```

### .gitignore のテンプレート

```
node_modules/
.env
.env.local
*.log
.DS_Store
```

### src/index.js の最初のサンプル

```javascript
console.log('Hello from Node.js!');
```

---

## ステップ4: 動作確認

```bash
npm start
```

`Hello from Node.js!` と表示されれば OK。

---

## ステップ5: 環境変数の管理(必要な場合)

API キーやパスワードを使う場合は、`.env` ファイルで管理する。

### dotenv のインストール

```bash
npm install dotenv
```

### .env ファイルの作成(リポジトリにコミットしない)

```
API_KEY=your_secret_key_here
DATABASE_URL=mongodb://...
```

### コードでの読み込み

```javascript
require('dotenv').config();

const apiKey = process.env.API_KEY;
console.log(apiKey);
```

`.env` は `.gitignore` に含まれているので、GitHub にはアップロードされない。
代わりに `.env.example` を作って、必要な環境変数の名前だけを共有する。

```
API_KEY=
DATABASE_URL=
```

---

## ステップ6: TypeScript を使う場合(オプション)

### TypeScript のインストール

```bash
npm install --save-dev typescript @types/node tsx
```

### tsconfig.json の作成

```bash
npx tsc --init
```

以下を推奨設定にする(主要部分のみ)。

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

### package.json のスクリプト更新

```json
"scripts": {
  "start": "tsx src/index.ts",
  "dev": "tsx --watch src/index.ts",
  "build": "tsc"
}
```

ファイル名を `src/index.js` から `src/index.ts` に変更する。

---

## ステップ7: よく使うパッケージ(用途別)

### HTTP リクエスト
```bash
npm install axios
```

### Excel ファイル操作
```bash
npm install xlsx
```

### CSV ファイル操作
```bash
npm install csv-parse csv-stringify
```

### MongoDB 接続
```bash
npm install mongodb
```

### 日付操作
```bash
npm install dayjs
```

---

## ステップ8: GitHub への push

```bash
git add .
git commit -m "初期セットアップ"
git push origin main
```

---

## トラブル対応

### `npm install` でエラーが出る場合
- Node.js のバージョンが古い可能性。`node --version` を確認し、20 以上に更新する。
- `package-lock.json` と `node_modules/` を削除してから再度 `npm install`。

### `.env` の内容が読み込まれない
- `require('dotenv').config()` がコードの一番上にあるか確認。
- `.env` ファイルがプロジェクトのルートにあるか確認(サブフォルダではダメ)。

### TypeScript で型エラーが大量に出る
- `npm install --save-dev @types/node` を実行して、Node.js の型定義をインストールする。
- 使っているライブラリごとに `@types/〇〇` をインストールする必要がある場合がある。

---

## 完了後の状態

- `npm start` でスクリプトが実行できる
- 環境変数を `.env` で安全に管理できる
- GitHub にコードを保存して履歴管理できる
- 必要に応じて TypeScript で書ける
