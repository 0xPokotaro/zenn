---
title: "NFT開発基礎知識（実装編）"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [NFT]
published: true
---
# 概要

### 環境要件

|項目|使用|説明|
|:--|:--|:--|
|PC|Mac|解説はMacを使用しています。|
|言語|Node.js v18以上|Hardhatを実行するために必要です。|
|エディター|VSCode|任意のエディターをご利用ください。|
|ウォレット|Metamask|デプロイ時にウォレットを使用します。|
|Node Providor|AllThatNode|デプロイする際に必要となります。|

### 開発の流れ

1. 開発準備
2. スマートコントラクトの実装
3. デプロイ準備
4. デプロイ
5. NFTをミント
6. OpenSeaで確認

# 1. 開発準備

#### 作業内容

開発するためのディレクトリを作成し、Hardhatのセットアップを行う。

## 1.1. 開発するためのディレクトリを作成

まずはターミナルを開きましょう。

![](/images/erc721-develop/1.png)

ターミナルを開いた後、現在の作業ディレクトリを確認します。

```bash
# 現在の作業場所が分かるコマンド
pwd
```

```bash
# 現在作業中のディレクトリのファイル一覧を表示するコマンド
ls
```

開発用のディレクトリ（フォルダ）を作成します。

```bash
# erc721-contractsディレクトリを作成するコマンド
mkdir erc721-contracts
```

```bash
# erc721-contractsディレクトリが作成されているか確認
ls
```

VSCodeから作成した「erc721-contracts」ディレクトリを開く

![](/images/erc721-develop/2.png)

## 1.2. Hardhatのセットアップ

VSCodeからターミナルを開く

![](/images/erc721-develop/3.png)

Hardhatのセットアップコマンドを実行

```bash
# Hardhatのセットアップコマンド
npx hardhat init
```

以下の画面が表示されるので、「Create a TypeScript projectを選択」を選択して、Enterを押す

```bash
(base) IT-06261-M:erc721-contracts kentaro.masuda$ npx hardhat init
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.22.2 👷‍

? What do you want to do? … 
  Create a JavaScript project
❯ Create a TypeScript project
  Create a TypeScript project (with Viem)
  Create an empty hardhat.config.js
  Quit
```

#### ルートディレクトリの設定

デフォルトで良いため、Enterを押す

```bash
👷 Welcome to Hardhat v2.22.2 👷‍

✔ What do you want to do? · Create a TypeScript project
? Hardhat project root: › /Users/0xpokotaro/Projects/0xpokotaro/erc721-contracts
```

#### .gitignoreの可否

デフォルト値は `.gitignore` を追加するため、そのまま、Enterを押す

```bash
👷 Welcome to Hardhat v2.22.2 👷‍

✔ What do you want to do? · Create a TypeScript project
✔ Hardhat project root: · /Users/kentaro.masuda/Projects/0xpokotaro/erc721-contracts
? Do you want to add a .gitignore? (Y/n) › y
```

#### ライブラリのインストール

ライブラリは必要となるため、Enterを押す

```bash
👷 Welcome to Hardhat v2.22.2 👷‍

✔ What do you want to do? · Create a TypeScript project
✔ Hardhat project root: · /Users/kentaro.masuda/Projects/0xpokotaro/erc721-contracts
✔ Do you want to add a .gitignore? (Y/n) · y
? Do you want to install this sample project's dependencies with npm (hardhat @nomicfoundation/hardhat-toolbox)? (Y/n) › y
```

ライブラリのインストールに少し時間がかかります、、

「Project created」が表示されるとセットアップ完了!!

```bash
✨ Project created ✨

See the README.md file for some example tasks you can run

Give Hardhat a star on Github if you're enjoying it! ⭐️✨

     https://github.com/NomicFoundation/hardhat
```

セットアップ完了!!

## 1.3. ERC721Aのインストール

VSCodeのターミナルから以下のコマンドを実行

```bash
npm install erc721a
```

## 1.4. ディレクトリ構成

```bash
erc721-contracts/
 ├ artifacts/        → コンパイル後に出力されたファイルを格納
 ├ cache/            → コンパイル時のキャッシュ
 ├ contracts/        → Solidityのファイルを格納
 ├ ignition/         → スクリプトを格納
 ├ node_modules/
 ├ test/             → テストファイルを格納
 ├ typechain-types/  → TypeScriptの型定義
 ├ .gitignore
 ├ hardhat.config.ts → Hardhatの設定ファイル
 ├ package-lock.json
 ├ package.json
 ├ README.md
 └ tsconfig.json
```

# 2. スマートコントラクトの実装

#### 作業内容

NFTのスマートコントラクトを作成する。

## 2.1. Solidityファイルを作成

contractsディレクトリ内に、`WaveHack.sol` を新規作成

## 2.1. スマートコントラクを実装

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import {ERC721A} from "erc721a/contracts/ERC721A.sol";

contract WaveHack is ERC721A {
    constructor() ERC721A("WaveHack", "WAVE") {}

    function _baseURI() internal pure override returns (string memory) {
        return "https://aquamarine-bright-chinchilla-232.mypinata.cloud/ipfs/QmZM2BCFzLPtWF9CntYQuPv7VCrMYEMT8mGaYA15U7on6X/";
    }

    function tokenURI(uint256 tokenId) public pure override returns (string memory) {
        return string(abi.encodePacked(_baseURI(), _toString(tokenId), ".json"));
    }

    function mint(uint256 quantity) public {
        _mint(msg.sender, quantity);
    }
}

```

# 3. デプロイ準備

#### 作業内容

実装したスマートコントラクトをEthereumネットワーク上にデプロイするためのスクリプトを作成し、Hardhatの設定ファイルに必要事項を追記する。

## 3.1. スクリプトファイルを作成

ignition/modulesディレクトリ内に、`WaveHack.ts` を作成

## 3.2. スクリプトを実装

```typescript
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

const WaveHackModule = buildModule("WaveHackModule", (m) => {
  const waveHack = m.contract("WaveHack");

  return { waveHack };
});

export default WaveHackModule;
```

## 3.3. Hardhatの設定

`hardhat.config.ts` を以下のように修正

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: "0.8.19",
  networks: {
    sepolia: {
      url: "AllThatNodeのURL",
      accounts: ["MetamaskのPrivateKey"],
    }
  },
  etherscan: {
    apiKey: "EtherScanのAPIKey"
  }
};

export default config;
```

## 4. デプロイ

#### 作業内容

作成したスマートコントラクトをEthereumネットワーク上にデプロイ、Verify & Publishを行う。

## 4.1. デプロイを実行

VSCodeのターミナルから以下のコマンドを実行

```bash
npx hardhat ignition deploy ./ignition/modules/WaveHack.ts --network sepolia --deployment-id sepolia-deployment 
```

以下が出力されることを確認

```bash
(base) IT-06261-M:erc721-contracts 0xpokotaro$ npx hardhat ignition deploy ./ignition/modules/WaveHack.ts --network sepolia
✔ Confirm deploy to network sepolia (11155111)? … yes
Compiled 1 Solidity file successfully (evm target: paris).
Hardhat Ignition 🚀

Deploying [ WaveHackModule ]

Batch #1
  Executed WaveHackModule#WaveHack

[ WaveHackModule ] successfully deployed 🚀

Deployed Addresses

WaveHackModule#WaveHack - 0xFD78cA4d210C06D4b656CC018Fe4d188C46a8c4b
```

## 4.2. Etherscanで確認

出力されたコントラクトアドレスをEtherscan(Sepolia)で検索

![](/images/erc721-develop/4.png)

## 4.3. Verify & Publish

以下のコマンドを実行して、Ethereum上にデプロイされたコードの検証と公開を行う。

```bash
npx hardhat ignition verify sepolia-deployment
```

以下が出力されると検証&公開が完了

```bash
(base) IT-06261-M:erc721-contracts 0xpokotaro$ npx hardhat ignition verify sepolia-deployment
Verifying contract "contracts/WaveHack.sol:WaveHack" for network sepolia...
Successfully verified contract "contracts/WaveHack.sol:WaveHack" for network sepolia:
  - https://sepolia.etherscan.io/address/0xBb50cec7998FF27010a79158D252b069775342a1#code
```

![](/images/erc721-develop/5.png)

コードが公開されていれば完了!!

# 5. NFTをミント

#### 作業内容

EtherscanからMetamaskを接続して、Mint関数を実行

## 5.1. ウォレット接続

「Write Contract」を選択して、「Connect to Web3」を選択

![](/images/erc721-develop/6.png)

注意事項が表示されるので「OK」を選択

> これはベータ版の機能であり、「現状のまま」および「利用可能な状態で」提供されることに注意してください。 Etherscan はいかなる保証も提供せず、この機能の継続使用による直接的または間接的な損失に対して責任を負いません。

![](/images/erc721-develop/7.png)

Metamaskを選択

![](/images/erc721-develop/8.png)

接続を選択

![](/images/erc721-develop/9.png)

「Connected」と表示されていればOK

![](/images/erc721-develop/10.png)

## 5.2. NFTをミント

mintの項目を選択し、発行したいNFTの枚数を入力

そして、「Write」を選択

![](/images/erc721-develop/11.png)

「View your transaction」を選択し、トランザクションを確認

![](/images/erc721-develop/12.png)

以下が表示されていればミント完了

![](/images/erc721-develop/13.png)

# 6. OpenSeaで確認

#### 作業内容

ミントしたNFTをOpenSeaで確認

## 6.1. Testnet用のOpenSeaにアクセス

Googleで「OpenSea testnet」で検索し、一番上に表示されたOpenSeaにアクセス

![](/images/erc721-develop/14.png)

ヒットしなかった場合は、以下リンクからアクセスしてください。

https://testnets.opensea.io/ja

画面右上の「ログイン」を選択して、Metamaskを接続

![](/images/erc721-develop/15.png)

Metamask接続後に、プロフィールを選択

![](/images/erc721-develop/16.png)

赤枠のNFTが発行されていればOK!!

![](/images/erc721-develop/17.png)
