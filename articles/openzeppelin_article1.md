---
title: "Solidity: ERC721のsafeMintでインクリメント"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Blockchain", "SmartContract", "Ethereum", "OpenZeppelin"]
published: true
---
## 概要

ERC721でミント関数を実装する際に、tokenIdをインクリメントする実装が必要となります。

OpenZeppelinで提供されているCountersというライブラリを使用して実装する方法を紹介いたします。

### Counters とは ？

インクリメント、デクリメント、またはリセットのみ可能なカウンターを提供するライブラリです。

```solidity
// @openzeppelin/contracts/utils/Counters.sol

library Counters {
    struct Counter {
        // This variable should never be directly accessed by users of the library: interactions must be restricted to
        // the library's function. As of Solidity v0.5.2, this cannot be enforced, though there is a proposal to add
        // this feature: see https://github.com/ethereum/solidity/issues/4637
        uint256 _value; // default: 0
    }

    // 現在のIDを返す
    function current(Counter storage counter) internal view returns (uint256) {
        return counter._value;
    }

    // 管理しているIDをインクリメントする
    function increment(Counter storage counter) internal {
        unchecked {
            counter._value += 1;
        }
    }

    // 管理しているIDをデクリメントする
    function decrement(Counter storage counter) internal {
        uint256 value = counter._value;
        require(value > 0, "Counter: decrement overflow");
        unchecked {
            counter._value = value - 1;
        }
    }

    // 管理しているIDをリセットする
    function reset(Counter storage counter) internal {
        counter._value = 0;
    }
}
```

## 実装

ERC721で実装例です。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
// カウンターをインポート
import "@openzeppelin/contracts/utils/Counters.sol";

contract MyToken is ERC721, Ownable {
    // カウンターstructをuse
    using Counters for Counters.Counter;

    // 状態変数 _tokenIdConter で、ライブラリを使用できるようにする
    Counters.Counter private _tokenIdCounter;

    constructor() ERC721("MyToken", "MTK") {}

    function safeMint(address to) public onlyOwner {
        // 現在のIDを取得
        uint256 tokenId = _tokenIdCounter.current();
        // インクリメント
        _tokenIdCounter.increment();
        // ミントを実行
        _safeMint(to, tokenId);
    }
}
```

## まとめ

インクリメントするだけの仕組みですが、コードレビュー済みのOpenZeppelinで提供されているライブラリを使用することをオススメいたします！
