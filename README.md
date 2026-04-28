# プロジェクトテンプレート

このリポジトリは、新規プロジェクト作成時の共通テンプレートです。

## 使い方

1. このリポジトリの「Use this template」から新規リポジトリを作成
2. 新しいリポジトリで Claude Code を起動
3. 作りたいプロジェクトの種類に応じて、Claude Code に以下のように指示する

## Claude Code への初回指示例
# 新規プロジェクト

このプロジェクトは [GAS / Firebase / Cloudflare / Node.js] を使います。
Claude Code は対応する `setup/〇〇_SETUP.md` を読んで初期セットアップを行ってください。


### GAS プロジェクトの場合
「`setup/GAS_SETUP.md` を読んで、GAS プロジェクトの初期セットアップを行ってください」

### Firebase Hosting プロジェクトの場合
「`setup/FIREBASE_SETUP.md` を読んで、Firebase Hosting の初期セットアップを行ってください」

### Cloudflare Workers プロジェクトの場合
「`setup/CLOUDFLARE_SETUP.md` を読んで、Cloudflare Workers の初期セットアップを行ってください」

### Node.js シンプルアプリの場合
「`setup/NODEJS_SETUP.md` を読んで、Node.js プロジェクトの初期セットアップを行ってください」

## ファイル構成

- `CLAUDE.md`: Claude Code が必ず参照する共通の開発ルール
- `setup/`: プロジェクト種別ごとの初期セットアップ手順
