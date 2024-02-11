---
title: "【スマートコントラクト】NFTプレセール機能の実装(Pillheads NFT)"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ethereum", "スマートコントラクト", "Solidity", "NFT", "ERC721"]
published: true
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

プレセールは、Phase1とPhase2の二段階で実施することができます。

##### プレセール仕様

- Phase1とPhase2の二段階でプレセールを実施することができます。
- Phase1とPhase2では、ミントできる数量が異なります。
- 参加資格がある場合、各Phaseにおいて一度だけミントすることができます。

##### プレセール機能一覧

1. プレセール価格設定
   - プレセール時のNFT価格を設定することができます。
2. プレセール期間設定
   - プレセール(Phase1)の開始日時を設定することができます。
   - プレセール(Phase2)の開始日時を設定することができます。
3. 参加資格設定
   - プレセール(Phase1)の参加資格を管理、検証することができます。
   - プレセール(Phase2)の参加資格を管理、検証することができます。
4. プレセール用ミント
   - プレセール(Phase1)でのミントが可能です。
   - プレセール(Phase2)でのミントが可能です。

# 2. スマートコントラクト

スマートコントラクトに実装されているプレセール機能を解説していきます。

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
// 初期値: プレセール(Phase1) UNIXタイムスタンプ
uint256 public presale1ActivationTime = 1655410800; // 2022年6月16日 20:20
// 初期値: プレセール(Phase2) UNIXタイムスタンプ
uint256 public presale2ActivationTime = 1655583600; // 2022年6月18日 20:20

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
ミント関数を行う前に、事前に登録されたMerkleRootを用いて、参加者資格を検証します。

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

    // amountの回数分、ミント処理
    for(uint i; i < amount;) {
        // 第一引数にNFTを受け取るアカウント、第二引数にTokenIDを指定してミントを実行
        // TokenID = MINTS_MAX - mintsLeft + 1
        _safeMint(msg.sender, MINTS_MAX - mintsLeft + 1);
        // ミントされたトークンの数を減らす（オーバーフローチェックを無効）
        // オーバーフローチェックを無効にすることでガス代を節約することができます
        unchecked { mintsLeft--; }
        // 次のトークンに進む（オーバーフローチェックを無効）
        unchecked { i++; }
    }

    // プレセール(Phase1)での再参加を防ぐためのフラグを設定
    unchecked { presale1Claimed[msg.sender] = true; }
}
```

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

    // mint their tokens
    for(uint i; i < amount;) {
        // 第一引数にNFTを受け取るアカウント、第二引数にTokenIDを指定してミントを実行
        // TokenID = MINTS_MAX - mintsLeft + 1
        _safeMint(msg.sender, MINTS_MAX - mintsLeft + 1);
        // ミントされたトークンの数を減らす（オーバーフローチェックを無効）
        unchecked { mintsLeft--; }
        // 次のトークンに進む（オーバーフローチェックを無効）
        unchecked { i++; }
    }

    // プレセール(Phase2)での再参加を防ぐためのフラグを設定
    unchecked { presale2Claimed[msg.sender] = true; }
  }
```

# 3. まとめ

今回は、スマートコントラクトに実装されているプレセール機能について詳しく解説しました。
プレセール機能はNFTやトークンの先行販売を効率的に管理するために重要で、限定的な販売やコミュニティの早期参加者への特典提供など、様々な目的で利用されます。

今回紹介した「Pillheads NFT」の例は、特別な条件を設定してアクセス権限を管理する方法を含め、プレセールの基本的な枠組みを提供しつつ、プロジェクト固有のニーズに応じたカスタマイズが可能であることを示しています。

このような技術的な詳細を理解し、プロジェクトに沿ったプレセール機能を実装していくことが大切です。
