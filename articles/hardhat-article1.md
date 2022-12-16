---
title: "HardHat: 導入方法"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Blockchain", "SmartContract", "Ethereum", "Hardhat"]
published: false
---
## Hardhat

EVM（Ethereum Virtual Machine）に対応したスマートコントラクトの開発環境ツールです。

公式：<https://hardhat.org/>

### Hardhatの特徴

- プラグインで拡張が可能
- TypeScript対応
- solidityファイル内でconsole.log()が使用可能

### Hardhatの導入方法

#### インストール

空のフォルダを作成して、Hardhatのインストールを行います。

```bash
# フォルダを作成
mkdir project

# フォルダに移動
cd project

# npmプロジェクトを作成
npm initoryarn init

# hardhatをインストール
npm install --save-dev hardhat
or
yarn add --dev hardhat
```

#### クイックスタート

```bash
# サンプルプロジェクトを作成
npx hardhat
```

以下が出力されるので、「Create a TypeScript project」を選択します。

```bash
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.9.9 👷‍

? What do you want to do? …
❯ Create a JavaScript project: JavaScriptに対応したサンプルプロジェクトを作成
  Create a TypeScript project: TypeScriptに対応したサンプルプロジェクトを作成
  Create an empty hardhat.config.js: 設定ファイルのみを作成
  Quit: 終了

```

#### ディレクトリ構成

以下が初期状態の構成です。

```bash
project
 ├ contracts/ Solidityファイルを設置するディレクトリ
 │  └ Lock.sol
 ├ scripts/   デプロイなどのスクリプトファイルを設置するディレクトリ
 │  └ deploy.sol
 ├ test/      テストスクリプトを設置するディレクトリ
 │  └ Lock.ts
 ├ hardhat.config.ts hardhatの設定ファイル
 ├ package.json
 ├ README.md
 └ tsconfig.json
```

#### 動作確認

```bash
# コンパイル
npx hardhat compile

# テスト
npx hardhat test

# デプロイ（ローカル環境）
# hardhatで構築されたNodeサーバー上にデプロイされます
npx hardhat npx hardhat run scripts/deploy.ts

# Nodeサーバー起動
npx hardhat node

# 起動したNodeサーバー上にデプロイ
npx hardhat run scripts/deploy.ts --network localhost
```
