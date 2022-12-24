---
title: "ERC20: TwitFiのスマコンを見てみた"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Blockchain", "SmartContract", "Ethereum", "仮想通貨"]
published: true
---
## 概要

TwitFiNFTのスマートコントラクトがどのような実装をされているか見ていきます。

### TwitFiとは ？

Tweet to Earn "TwitFi"

TwitFiは、ツイートをしてBirdNFTを育てて、TWTトークンを稼いでいく新感覚のGameFiです。

公式 <https://twitfi.com/>

TwitFi (ERC20) <https://etherscan.io/token/0xd4df22556e07148e591b4c7b4f555a17188cf5cf#code>

TwitFiNFT (ERC721) <https://etherscan.io/token/0x94cce07f299945cfe80e309c85cb0a784b3ee6c2#code>

### TwitFi

Compiler Version 0.8.17

### インポートされているライブラリ

- @openzeppelin/contracts/access/Ownable.sol
- @openzeppelin/contracts/security/Pausable.sol
- @openzeppelin/contracts/utils/math/SafeMath.sol

### 関連アドレス一覧

- TwitFi(TWT): 0xd4Df22556e07148e591B4c7b4f555a17188CF5cF
- TwitFiオーナー: 0x2340accae6c964059e0ffcf9e2bb4fca9cfde653
- Wrapped Ether(WETH): 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2

### 関数一覧

書き込みしている関数にフォーカスしてピックアップしました。

#### onlyOwner

コントラクトの所有者(※1)のみ実行ができる関数。

- pause: 送金をストップする関数
- unpause: 送金をスタートする関数
- mint: TWTをミントする関数
- addPairs: ペアを追加、削除する関数
- setLiquidityFeePercent: 流動性手数料を設定する関数
- setBurnFee: 破棄する手数料を設定する関数
- manualswap: コントラクト上で保持しているTWTを全てETHにスワップする関数
- manualBurn: TWTを破棄する関数
- openTrading: トレードを開始する関数
- withdraw: コントラクト上で保持しているETHを全てオーナーに送金する関数

#### private

- swapAndLiquify: コントラクト上で保持している残高からETHにスワップしてLPトークンを生成する関数
- swapTokensForEth
  - 内部で呼び出している関数: [swapExactTokensForETHSupportingFeeOnTransferTokens](https://github.com/Uniswap/v2-periphery/blob/0335e8f7e1bd1e8d8329fd300aea2ef2f36dd19f/contracts/UniswapV2Router02.sol#L379)
- addLiquidity: TWTとETHのLPトークンを生成する関数

#### internal

- _transfer: TWTを送金処理する際の内部関数
- _beforeTokenTransfer: TWTをmint,burnを処理する際の内部関数

※1. コントラクト作成時に使用されたアカウントがオーナーとなる。

[コントラクト作成時のトランザクション](https://etherscan.io/tx/0x506ffb8e80724507fd87f3de42e7e2939655748171e25b762297944659156905)：Fromが当該コントラクトのオーナーです。
