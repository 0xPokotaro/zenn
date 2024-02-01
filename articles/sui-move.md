---
title: "sui"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## Sui Move

https://docs.sui.io/concepts/sui-move-concepts

### 概念

Moveは、オンチェーンオブジェクト（スマートコントラクト）を扱うためのパッケージを作成するオープンソース言語です。

Suiのオブジェクトシステムは、Moveに新機能と新たな制約を加えた拡張が実装されており、これは"Sui Move"とも呼ばれています。

Sui Moveで使用できない機能は以下の二点です。

- [Global Storage Operators](https://move-language.github.io/move/global-storage-operators.html)
- [Key Abilities](https://github.com/move-language/move/blob/main/language/documentation/book/src/abilities.md)

### MoveとSui Moveの主な違い

- Suiは独自のオブジェクト中心のグローバルストレージを使用します。
- アドレスはオブジェクトIDを表します。
- Suiオブジェクトにはグローバルに一意のIDがあります。
- Suiにはモジュール初期化子 (init) があります。
- Suiエントリーポイントはオブジェクト参照を入力として受け取ります。

### オブジェクト中心のグローバル

作成中

### Object Model

Suiにおけるストレージの基本単位は"オブジェクト"。

Suiのストレージは、一位のIDによってオンチェーンでアドレス指定できるオブジェクトを中心としている。

スマートコントラクトは、オブジェクトで、Suiネットワーク上のオブジェクトを操作する。

- Sui Move Package: Sui Moveのバイトコードモジュールのセット。
- Sui Move Object: Sui Move Packageで生成される具体的なデータオブジェクト。

### 参考文献

Move book
https://move-language.github.io/move/introduction.html

