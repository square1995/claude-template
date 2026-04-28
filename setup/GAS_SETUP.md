# GAS プロジェクト初期セットアップ手順

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
