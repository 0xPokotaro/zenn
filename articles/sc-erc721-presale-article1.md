---
title: "【スマートコントラクト】NFTプレセール機能の実装"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ethereum", "スマートコントラクト"]
published: false
---

# 1. はじめに

本記事では、「Pillheads NFT」というNFTプロジェクトに実装されたスマートコントラクトのプレセール機能について解説します。
NFTプロジェクト自体の詳細については割愛します。

##### 参考文献

- [Pillheads](https://www.pillheads.art/)
- [OpenSea - Pillheads NFT](https://opensea.io/ja/collection/pillheadsnft)
- [Etherscan - PILLHEADS](https://etherscan.io/address/0x1b23d0f0f6dc3547c1b6945152acbfd6eaad85b0#code)

## 1.2. プレセールとは

NFTのプレセールは、公式販売前に特定グループに先行してNFTを低価格で提供する期間です。
プレセールを行うことで、プロジェクトへの早期支持を集め、資金調達とコミュニティ構築を目指します。

## 1.3. Pillheads NFTのプレセール機能

1. プレセール価格設定
   - `setPresalePrice()` : プレセールの価格を設定する関数
2. プレセール期間設定
   - `setPresale1ActivationTime()`: プレセール(Phase1)の開始日時を設定する関数
   - `setPresale2ActivationTime()`: プレセール(Phase2)の開始日時を設定する関数
3. 参加資格設定
   - `setPresale1MerkleRoot()`: プレセール(Phase1)の参加資格を検証するMerkleRootを設定する関数
   - `setPresale2MerkleRoot()`: プレセール(Phase2)の参加資格を検証するMerkleRootを設定する関数
4. プレセール用ミント
   - `mintPresale1()`: プレセール(Phase1)中にミントをする関数
   - `mintPresale2()`: プレセール(Phase2)中にミントをする関数

# 2. スマートコントラクト

## 2.1. プレセール価格設定

`setPresalePrice`関数を使用して、プレセール時のNFT価格を設定できます。
また関数は、管理者のみが実行できるように制御されています。
`presalePrice`変数には、初期値が設定されています。

```solidity
/// 初期価格: 0.1ETH (100000000000000000 Wei)
uint256 public presalePrice = 100000000000000000;

/// @dev プレセールの価格を設定する関数
/// @param price 新しいプレセール価格(Wei単位)
function setPresalePrice(uint256 price) external onlyAdmin {
    presalePrice = price;
}
```

## 2.2. プレセール期間設定

`setPresale1ActivationTime`関数と`setPresale2ActivationTime`関数を使用して、プレセールの開始時刻を設定できます。
また関数は、管理者のみが実行できるように制御されています。
`presale1ActivationTime`変数と`presale2ActivationTime`変数には、初期値が設定されています。

```solidity
// 初期値: プレセール(Phase1) 2022年6月16日 20:20
uint256 public presale1ActivationTime = 1655410800;
// 初期値: プレセール(Phase2) 2022年6月18日 20:20
uint256 public presale2ActivationTime = 1655583600;

/// @dev プレセール(Phase1)の開始時刻を設定する関数
/// @param timestampInSeconds 新しい開始時刻(UNIXタイムスタンプ)
function setPresale1ActivationTime(uint256 timestampInSeconds) external onlyAdmin {
    presale1ActivationTime = timestampInSeconds;
}

/// @dev プレセール(Phase2)の開始時刻を設定する関数
/// @param timestampInSeconds 新しい開始時刻(UNIXタイムスタンプ)
function setPresale2ActivationTime(uint256 timestampInSeconds) external onlyAdmin {
    presale2ActivationTime = timestampInSeconds;
}
```

## 2.3. 参加資格設定

参加資格は、OpenZeppelinのMerkleProofという仕組みを利用して管理されています。
この仕組みを使用することで、スマートコントラクト側で参加者リストを保持する必要がなく、bytes32型の文字列(通称: MerkeRoot)を保持するだけで資格の検証が可能です。

https://docs.openzeppelin.com/contracts/5.x/api/utils#MerkleProof

`setPresale1MerkleRoot`関数と`setPresale2MerkleRoot`関数を使用して、プレセールの参加資格を検証するMerkleRootを設定できます。
また関数は、管理者のみが実行できるように制御されています。

```solidity
/// 初期値: プレセール(Phase1)
bytes32 private presale1Root = bytes32(0);
/// 初期値: プレセール(Phase2)
bytes32 private presale2Root = bytes32(0);

/// @dev プレセール(Phase1)の参加資格を設定する関数
/// @param newRoot 新しいMerkleRoot
function setPresale1MerkleRoot(bytes32 newRoot) external onlyAdmin {
    presale1Root = newRoot;
}

/// @dev プレセール(Phase1)の参加資格を設定する関数
/// @param newRoot 新しいMerkleRoot
function setPresale2MerkleRoot(bytes32 newRoot) external onlyAdmin {
    presale2Root = newRoot;
}
```

## 2.4. プレセール用ミント

`mintPresale1`関数と`mintPresale2`関数を使用して、プレセール期間にミントを行えます。

```solidity
/// プレセール(Phase1)のミント関数
/// @param maxMints 最大ミント数量
/// @param sigV 署名
/// @param sigR 署名
/// @param sigS 署名
/// @param merkleProof プレセールの参加資格を検証するためのMerkleProof
/// @param amount ミントする数量
function mintPresale1(
    uint256 maxMints,
    uint8 sigV,
    bytes32 sigR,
    bytes32 sigS,
    bytes32[] calldata merkleProof,
    uint256 amount
)
    public
    payable
{
    require(presale1Activated(), "TooEarly");
    require(amount <= mintsLeft, "ExceedsSupply");
    require(presalePrice * amount == msg.value, "BadPrice");
    require(verify(msg.sender, sigV, sigR, sigS) == true, "BadSignature");
    require(checkPresale1MerkleProof(merkleProof, maxMints), "BadProof");
    require(amount <= maxMints, "ExceedsPresaleMax");
    require(!presale1Claimed[msg.sender], "Claimed");

    // amountの回数分、ミント処理
    for(uint i; i < amount;) {
        // 第一引数にNFTを受け取るアカウント、第二引数にTokenIDを指定してミントを実行
        // TokenID = MINTS_MAX - mintsLeft + 1
        _safeMint(msg.sender, MINTS_MAX - mintsLeft + 1);
        // ミントされたトークンの数を減らす（オーバーフローチェックを無効）
        unchecked { mintsLeft--; }
        // 次のトークンに進む（オーバーフローチェックを無効）
        unchecked { i++; }
    }

    // プレセール(Phase1)での再参加を防ぐためのフラグを設定
    unchecked { presale1Claimed[msg.sender] = true; }
}
```

### バリデーション

```solidity
// プレセール(Phase1)が開始されていない場合はエラー
require(presale1Activated(), "TooEarly");
// ミント数がTotalSupplyを超えている場合はエラー
require(amount <= mintsLeft, "ExceedsSupply");
// 支払い金額が販売金額と一致しない場合はエラー
require(presalePrice * amount == msg.value, "BadPrice");
// 署名が有効でない場合はエラー
require(verify(msg.sender, sigV, sigR, sigS) == true, "BadSignature");
// プレセール(Phase1)のMerkleProofが一致しない場合はエラー
require(checkPresale1MerkleProof(merkleProof, maxMints), "BadProof");
// ミント数が最大ミント数以上の場合はエラー
require(amount <= maxMints, "ExceedsPresaleMax");
// 既にミント済みアカウントの場合はエラー
require(!presale1Claimed[msg.sender], "Claimed");
```

# 関数: mintPresale2

```solidity
/// プレセール(Phase2)のミント関数
/// @param sigV 署名
/// @param sigR 署名
/// @param sigS 署名
/// @param merkleProof プレセールの参加資格を検証するためのMerkleProof
/// @param amount ミントする数量
function mintPresale2(
    uint8 sigV,
    bytes32 sigR,
    bytes32 sigS,
    bytes32[] calldata merkleProof,
    uint256 amount
)
    public
    payable
{
    require(presale2Activated(), "TooEarly");
    require(amount <= mintsLeft, "ExceedsSupply");
    require(presalePrice * amount == msg.value, "BadPrice");
    require(verify(msg.sender, sigV, sigR, sigS) == true, "BadSignature");
    require(checkPresale2MerkleProof(merkleProof), "BadProof");
    require(amount <= 2, "ExceedsPresaleMax");
    require(!presale2Claimed[msg.sender], "Claimed");

    // mint their tokens
    for(uint i; i < amount;) {
      // reduce the total pop after minting
      _safeMint(msg.sender, MINTS_MAX - mintsLeft + 1);
      unchecked { mintsLeft--; }
      unchecked { i++; }
    }

    // lock them out of re-entering presale 2
    unchecked { presale2Claimed[msg.sender] = true; }
  }
```

### 修飾子

```solidity
public /// 公開関数
payable /// Etherを送金することができる
```

### バリデーション

```solidity
// プレセール(Phase2)が開始されていない場合はエラー
require(presale2Activated(), "TooEarly");
// ミント数がTotalSupplyを超えている場合はエラー
require(amount <= mintsLeft, "ExceedsSupply");
// 支払い金額が販売金額と一致しない場合はエラー
require(presalePrice * amount == msg.value, "BadPrice");
// 署名が有効でない場合はエラー
require(verify(msg.sender, sigV, sigR, sigS) == true, "BadSignature");
// プレセール(Phase2)のMerkleProofが一致しない場合はエラー
require(checkPresale2MerkleProof(merkleProof), "BadProof");
// ミント数が2以上の場合はエラー
require(amount <= 2, "ExceedsPresaleMax");
// 既にミント済みアカウントの場合はエラー
require(!presale2Claimed[msg.sender], "Claimed");
```
