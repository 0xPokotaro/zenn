---
title: "【スマートコントラクト】NFTプレセール機能の実装(YuGiYn)"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Ethereum", "スマートコントラクト", "Solidity", "NFT", "ERC721"]
published: false
---

# 1. はじめに

本記事では、「YuGiYn」というNFTプロジェクトに実装されたスマートコントラクトのプレセール機能について解説します。
NFTプロジェクト自体の詳細については割愛します。

##### 参考文献

- [YuGiYn](https://yu-gi-yn.com/)
- [OpenSea - YuGiYn](https://opensea.io/ja/collection/yugiyn-official)
- [Etherscan - YuGiYn](https://etherscan.io/address/0x477f885f6333317f5b2810ecc8afadc7d5b69dd2#code)

## 1.2. プレセールとは

NFTのプレセールは、公式販売前に特定グループに先行してNFTを低価格で提供する期間です。
プレセールを行うことで、プロジェクトへの早期支持を集め、資金調達とコミュニティ構築を目指します。

## 1.3. YuGiYnのプレセール機能

NFTのミントセールは、三段階で実施することができます。

##### 各セール仕様

- Pre-sale, Whitelist sale, Public sale, Post-saleの四段階で実施することができます。

##### セール機能一覧
