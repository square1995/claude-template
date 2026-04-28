markdown# GAS プロジェクト初期セットアップ手順

このドキュメントは、Google Apps Script (GAS) プロジェクトを clasp + GitHub Actions で
自動デプロイする環境を構築するための手順です。

Claude Code は、この手順に従って初期セットアップを行ってください。
各ステップでユーザーに確認を取りながら進めてください。

---

## 前提条件の確認

まずユーザーに以下を確認してください。

1. GAS プロジェクトは既に作成済みか?(URL から Script ID を取得できる状態か)
2. Google Cloud Platform (GCP) プロジェクトと連携済みか?
3. clasp を初めて使うか?

---

## ステップ1: ローカル(または Claude Code 環境)で clasp 認証

Claude Code はユーザーに以下のコマンドを案内し、実行してもらう。

```bash
npm install -g @google/clasp
clasp login
```

`clasp login` はブラウザが開いて Google 認証を求めるため、ユーザー側で実施する必要がある。

認証が完了すると `~/.clasprc.json` に認証情報が保存される。
このファイルの中身は GitHub Secrets に登録するので、ユーザーに**中身をコピーして渡してもらう**。

---

## ステップ2: プロジェクト構造の作成

リポジトリのルートに以下のファイル・フォルダを作成する。
.
├── .clasp.json            ← Script ID を記載
├── .claspignore           ← clasp push 時に無視するファイル
├── appsscript.json        ← GAS のマニフェスト
├── src/                   ← GAS のソースコード(.js / .html)
└── .github/
└── workflows/
└── deploy.yml     ← GitHub Actions の設定

### .clasp.json のテンプレート

```json
{
  "scriptId": "ここにユーザーから受け取った Script ID を入れる",
  "rootDir": "./src"
}
```

ユーザーに GAS プロジェクトの URL を教えてもらい、URL の `/d/〇〇〇/edit` の `〇〇〇` 部分が Script ID。

### .claspignore のテンプレート
全部無視してから、必要なものだけ許可
/
clasp 関連は除外しない
!appsscript.json
!src//*.js
!src//.html
!src/**/.gs
不要なものは無視
node_modules/**
.git/**
.github/**
*.md
.clasp.json
.claspignore

### appsscript.json のテンプレート

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

---

## ステップ3: GitHub Actions の設定

`.github/workflows/deploy.yml` に以下を作成する。

```yaml
name: Deploy to GAS

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

      - name: Install clasp
        run: npm install -g @google/clasp

      - name: Setup clasp credentials
        run: echo '${{ secrets.CLASPRC_JSON }}' > ~/.clasprc.json

      - name: Push to GAS
        run: clasp push --force
```

---

## ステップ4: GitHub Secrets の登録

以下をユーザーに案内する。

1. GitHub のリポジトリページを開く
2. **Settings** → **Secrets and variables** → **Actions**
3. **New repository secret** をクリック
4. Name: `CLASPRC_JSON`
5. Secret: ステップ1で取得した `~/.clasprc.json` の中身を**そのまま貼り付け**
6. **Add secret** をクリック

---

## ステップ5: 動作確認

1. 何か小さな変更をして main ブランチにコミット&プッシュ
2. GitHub の Actions タブを開いて、ワークフローが成功するか確認
3. GAS エディタでコードが反映されているか確認

---

## トラブル対応

### clasp push が認証エラーで失敗する場合
- `~/.clasprc.json` の中身が古くなっている可能性。ユーザーに `clasp login` を再実行してもらい、新しい中身を Secrets に再登録する。

### Script ID が間違っている場合
- `.clasp.json` の `scriptId` を確認。GAS の URL から正しい ID を取得し直す。

### appsscript.json が反映されない場合
- GAS エディタの「プロジェクトの設定」で「'appsscript.json' マニフェスト ファイルをエディタで表示する」を ON にする。

---

## 完了後の状態

- main ブランチに push するだけで GAS に自動デプロイされる
- ローカルで `clasp pull` すれば GAS 側の変更を取得できる
- GitHub でコード履歴を管理できる
