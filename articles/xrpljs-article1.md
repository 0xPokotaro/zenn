---
title: "XRPL: 初心者向け「xrpl.js」入門"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["XRPL", "xrpljs", "web3", "blockchain"]
published: false
---

## 1. はじめに

### 1.1. xrpl.jsとは何か？

xrpl.jsは、JavaScript/TypeScriptでXRP Ledgerを操作するためのライブラリです。

このライブラリを使用することで、XRP Ledger上でトランザクションを生成、署名、送信し、その他の多くの機能を実行することが可能になります。

### 1.2. この記事で学べること

- xrpl.jsのセットアップと使用方法
- 基本的なトランザクションの作成と管理
- 応用機能の実装
- セキュリティと最適化
- 実例と実践

## 1. 基礎知識

### 1.1. ネットワークの種類

これらのテストネットワークは、実際の資金を使用せずにXRP Ledgerのテストを行うことができます。

|ネットワーク|WebSocket|説明|
|:---|:---|:--|
|Testnet|wss://s.altnet.rippletest.net:51233/|メインネット同等|
|Devnet|wss://s.devnet.rippletest.net:51233/|今後の修正予定の機能を反映。|
|Xahau-Testnet|wss://xahau-test.net/|フックすが有効なテストネット|

#### XRP Faucets

XRP Ledgerテストネットワークのトークンを発行できるサイト

https://xrpl.org/ja/xrp-testnet-faucet.html

### 1.2. アドレスの種類

#### PublicKey (公開鍵)

誰にでも後悔されているアカウント(ウォレット)を識別するための一意のコード。
アカウント(ウォレット)で署名する際に使用します。

サンプル値: `ED1AA872636AD6ED618AAECB7F387F2234155A28FD89193DE344B6CD4BC4B76C1A`

#### PrivateKey (秘密鍵)

PublicKeyとペアになっているウォレットのパスワード。他人に知られてはいけない。
アカウント(ウォレット)にアクセスするために使用します。

サンプル値: `ED2A2DD166BB2A391BFC76B841A57AC019562FCF8AF5CE22315193A2BBC7688D7C`

#### ClassicAddress (ウォレット)

公開鍵から生成された、XRP Ledger上のアカウントのアドレス。
XRPやトークンを送るときに使用します。

サンプル値: `rUSE6ZjXKnWDajT8r51a49iwC7pA5Lq5Kg`

## 3. 初期設定

### 3.1. 要件

- Node.js v16 推奨

### 3.2. インストール

まず最初にxrpl.jsをインストールします。

```bash
# npmの場合
$ npm install --save xrpl

# yarnの場合
$ yarn add xrpl
```

インポートをすることで、ライブラリを使用することができます。

```typescript
import xrpl from xrpl
```

## 4. Class Client

rippledサーバーと対話するためのクライアントです。

### 4.1. 使い方

#### 構文

```typescript
new Client(server, options?): Client
```

#### パラメータ

- server: `string`
  - 接続するサーバーのURL
- options?: `ClientOptions = {}`
  - クライアント設定のオプション: https://js.xrpl.org/interfaces/ClientOptions.html

#### 使用例

```typescript
import { Client } from xrpl

async function main() {
  const client = new Client('wss://s.altnet.rippletest.net:51233')
  await client.connect()
  // 処理
  client.disconnect()
}
```

### 4.2. 新規ウォレット作成

#### fundWallet関数

XRPLのテストネット用のユーティリティ関数で、新しいウォレットを作成し、テスト用のXRPを付与する機能を提供します。

```typescript
import { Client } from xrpl

async function main() {
  const client = new Client('wss://s.altnet.rippletest.net:51233')
  await client.connect()

  const fundWallet = await client.fundWallet()
  console.log(fundWallet)

  client.disconnect()
}
```

fundWalletのレスポンス


```javascript
{
  wallet: Wallet {
    publicKey: 'ED1AA872636AD6ED618AAECB7F387F2234155A28FD89193DE344B6CD4BC4B76C1A',
    privateKey: 'ED2A2DD166BB2A391BFC76B841A57AC019562FCF8AF5CE22315193A2BBC7688D7C',
    classicAddress: 'rUSE6ZjXKnWDajT8r51a49iwC7pA5Lq5Kg',
    seed: 'sEdS6UP2W1h6gnQ4vVkHuBTpa4ChSXD'
  },
  balance: 10000
}
```


### fundWallet

## トランザクションの処理

### トランザクションの作成と送信

### トランザクションのステータス確認

## 応用機能の利用

### スマートコントラクトの基本

### NFTとカスタムトークン

## 実際のアプリケーション例

### 簡単な支払システムの構築

### XRPLを利用したDAppの開発

## まとめ

### 学習のまとめ
