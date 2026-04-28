# Firebase Hosting プロジェクト初期セットアップ手順

このドキュメントは、Firebase Hosting に静的サイトを GitHub Actions で
自動デプロイする環境を構築するための手順です。

Claude Code は、この手順に従って初期セットアップを行ってください。
各ステップでユーザーに確認を取りながら進めてください。

---

## 前提条件の確認

まずユーザーに以下を確認してください。

1. Firebase プロジェクトは既に作成済みか?
2. プロジェクト ID は何か?(Firebase コンソールの「プロジェクトの設定」で確認)
3. Hosting を有効化済みか?
4. firebase-tools(firebase CLI)を初めて使うか?

---

## ステップ1: firebase CLI のインストールとログイン

ユーザーに以下のコマンドを案内する。

```bash
npm install -g firebase-tools
firebase login
```

`firebase login` はブラウザが開いて Google 認証を求めるため、ユーザー側で実施する必要がある。

---

## ステップ2: プロジェクトの初期化

リポジトリのルートで以下を実行するようユーザーに案内する。

```bash
firebase init hosting
```

対話形式で以下を選択してもらう。

- **Use an existing project** → 該当する Firebase プロジェクトを選ぶ
- **Public directory**: `public`(基本これでOK)
- **Configure as a single-page app**: アプリの種類による(SPAなら Yes、複数ページなら No)
- **Set up automatic builds and deploys with GitHub**: **Yes** を選ぶ(これで GitHub Actions が自動設定される)
- リポジトリ名を聞かれたら、対象リポジトリ名を入力(例: `square1995/myapp`)
- **Set up the workflow to run a build script before every deploy**: ビルドが必要なら Yes、純粋な HTML/CSS/JS なら No
- **Set up automatic deployment to your site's live channel when a PR is merged**: **Yes**(main ブランチへのマージで自動デプロイ)
- **What is the name of the GitHub branch associated with your site's live channel**: `main`

これで以下のファイルが自動作成される。

```
.
├── .firebaserc                    ← プロジェクト ID
├── firebase.json                  ← Hosting 設定
├── public/                        ← 公開するファイル(index.html など)
└── .github/
    └── workflows/
        ├── firebase-hosting-merge.yml         ← main マージ時の本番デプロイ
        └── firebase-hosting-pull-request.yml  ← PR 時のプレビューデプロイ
```

GitHub Secrets にもサービスアカウントキーが自動登録される。

---

## ステップ3: ファイル構成の確認

`public/` フォルダに `index.html` などの公開ファイルを配置する。

シンプルな例:

```
public/
├── index.html
├── style.css
└── script.js
```

---

## ステップ4: 動作確認

### ローカルでの確認

```bash
firebase serve
```

ブラウザで `http://localhost:5000` を開いて、サイトが表示されるか確認。

### 本番デプロイの確認

1. main ブランチに push
2. GitHub の Actions タブでワークフローが成功するか確認
3. `https://〇〇〇.web.app` または `https://〇〇〇.firebaseapp.com` にアクセスして確認

---

## トラブル対応

### `firebase init` で「No projects found」と表示される場合
- `firebase login` が正しく完了していない可能性。再度ログインを試す。
- もしくは複数の Google アカウントを使っている場合、`firebase logout` してから正しいアカウントで `firebase login` する。

### GitHub Actions のデプロイが失敗する場合
- Settings → Secrets で `FIREBASE_SERVICE_ACCOUNT_〇〇〇` という Secret が登録されているか確認。
- `firebase init hosting` の途中で GitHub 連携設定を飛ばしてしまった場合は、再度 `firebase init hosting:github` を実行する。

### デプロイは成功したのに古い内容が表示される場合
- ブラウザのキャッシュ。Ctrl + Shift + R(Mac は Cmd + Shift + R)で強制リロード。
- それでも変わらなければ、Firebase コンソールの Hosting 画面で最新のデプロイが反映されているか確認。

### カスタムドメインを使いたい場合
- Firebase コンソールの Hosting 画面 → 「カスタムドメインを追加」
- DNS 設定(A レコード)が必要なので、ドメインプロバイダ側でも設定する。

---

## 完了後の状態

- main ブランチに push するだけで Firebase Hosting に自動デプロイされる
- PR を作成すると、自動で「プレビュー URL」が生成される
- 本番反映前に動作確認できる
