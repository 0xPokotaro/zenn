---
title: "プレセール"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# プレセールとは

# コード全体

```solidity
pragma solidity ^0.8.23;

contract Hoge is ERC721Burnable {
    // Merkle Tree root hashes for happylist presale phases
    bytes32 private presale1Root = bytes32(0);
    bytes32 private presale2Root = bytes32(0);

    mapping(address => bool) public presale1Claimed;
    mapping(address => bool) public presale2Claimed;

    constructor() ERC721("Pillheads", "PILLHEADS") {
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    function setPresalePrice(uint256 price) external onlyAdmin {
        presalePrice = price;
    }

    function setSalePrice(uint256 price) external onlyAdmin {
        salePrice = price;
    }

    /// この最初の 5 つのパラメータは、呼び出しを検証するために使用されます。
    /// 最大 mint、v、r、s、またはマークル証明が間違っている場合、これは元に戻ります。
    /// それらがすべて正しい場合、ユーザーは最大ミントまで <= をミントできます。
    /// その後、このプレセールでは二度と鋳造できなくなります。
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
        // プリセール1フェーズが開始されていることを確認
        require(presale1Activated(), "TooEarly");
        // ミントしようとしている量が残りの供給量を超えていないことを確認
        require(amount <= mintsLeft, "ExceedsSupply");
        // 送られてきたETHの量が、ミントするトークンの総価格と一致することを確認
        require(presalePrice * amount == msg.value, "BadPrice");
        // 送信者の署名が有効であることを確認
        require(verify(msg.sender, sigV, sigR, sigS) == true, "BadSignature");
        // メルクル証明が有効で、ユーザーがプリセール1に参加できることを確認
        require(checkPresale1MerkleProof(merkleProof, maxMints), "BadProof");
        // ユーザーがミントしようとしている量が、指定された最大ミント数を超えていないことを確認
        require(amount <= maxMints, "ExceedsPresaleMax");
        // ユーザーが既にプリセール1でミントを行っていないことを確認
        require(!presale1Claimed[msg.sender], "Claimed");

        // トークンをミントする処理
        for(uint i; i < amount;) {
            // トークンを安全にミントし、送信者に送る
            _safeMint(msg.sender, MINTS_MAX - mintsLeft + 1);
            // ミントされたトークンの数を減らす（オーバーフローチェックを無効）
            unchecked { mintsLeft--; }
            // 次のトークンに進む（オーバーフローチェックを無効）
            unchecked { i++; }
        }

        // ユーザーがプリセール1での再参加を防ぐためのフラグを設定
        unchecked { presale1Claimed[msg.sender] = true; }
    }
}
```
