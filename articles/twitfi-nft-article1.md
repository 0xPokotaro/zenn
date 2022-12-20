---
title: "ERC721: TwitFiNFTのスマコンを見てみた"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Blockchain", "SmartContract", "Ethereum", "NFT"]
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

### TwitFiNFT

Compiler Version 0.8.17

### imports

- ERC721
- ERC721Enumerable: ERC721の拡張版
- Ownable: アクセス制御
- AccessControl: アクセス制御
- DefaultOperatorFilterer: 

### Struct

トークンに紐づけられている情報がSolidity内で構造体 (Struct) として定義されています。

- tokenList
  - creator: mintをしたアドレス (ミント時に設定される)
  - id: トークンID (ミント時に設定される)
  - issueDate: 発行日時 Unixtime
  - modifyDate: 属性情報が更新された日時 Unixtime
  - tranferCount: 転送回数 (transferされると 1 増加する)
  - _type: TwiFiNFTプロジェクトで設定しているNFTタイプ

### Functions

#### tokenInfo

[Etherscan - tokenInfo](https://etherscan.io/token/0x94cce07f299945cfe80e309c85cb0a784b3ee6c2#readContract) から、調べたいトークンIDを入力してQueryボタンを押すと、トークンに紐づけられている情報が確認できます。

```solidity

/// @dev トークン(NFT)に紐づけられている属性情報を返却する関数
/// @param tokenId 調べたいトークンID
function tokenInfo(uint256 tokenId) public view existingToken(tokenId) returns (TokenList memory) {
    TokenList memory token = _listTokens[tokenId];
    return token;
}
```

#### mint

鳥NFTをミントした時に実行される関数です。

```solidity
/// @dev NFTを新しく発行する関数
/// @param _receiver 新しく発行したNFTを受け取るアドレス
/// @param _type トークンに紐づけるタイプ
/// @param _tokenId 新しく発行するトークンID
function mint(address _receiver, string memory _type, uint256 _tokenId) external onlyMinter {
    _mintToken(_receiver, _type, _tokenId);
}
```

#### bulkMint

```solidity
/// @dev 一括でNFTを新しく発行する関数
/// @param _tos 新しく発行したNFTを受け取るアドレスのリスト
/// @param _types トークンに紐づけるタイプのリスト
/// @param _tokenIds 新しく発行するトークンIDのリスト
function bulkMint(address[] memory _tos, string[] memory _types, uint256[] memory _tokenIds) external onlyMinter {
    uint8 i;
    for (i = 0; i < _tos.length; i++) {
        _mintToken(_tos[i], _types[i], _tokenIds[i]);
    }
}
```

#### burn

```solidity
/// @dev トークンID (NFT) を破棄する関数
/// @param tokenId 破棄するトークンID
function burn(uint256 tokenId) public virtual {
    require(_isApprovedOrOwner(_msgSender(), tokenId), "burn caller is not owner nor approved");
    _burn(tokenId);
}
```

#### withdraw

オーナー (TwitFiプロジェクト運営)しか実行できない関数です。

```solidity
/// @dev コントラクト上にステーキングしているトークンを回収する関数
function withdraw() public onlyOwner {
    uint amount = address(this).balance;
    require(amount > 0, "Insufficient balance");
    (bool success, ) = payable(owner()).call {
        value: amount
    }("");

    require(success, "Failed to send Matic");
}
```

#### Approve

onlyAllowedOperatorApprovalが追加されている。

[ProjectOpenSea/operator-filter-registry](https://github.com/ProjectOpenSea/operator-filter-registry#filtered-addresses) OpenSeaが提供しているブラックリストをフィルターする機能。

登録されているアドレス宛には実行できなくすることができる。

```solidity
function setApprovalForAll(address operator, bool approved) public override(ERC721, IERC721) onlyAllowedOperatorApproval(operator) {
    super.setApprovalForAll(operator, approved);
}

function approve(address operator, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperatorApproval(operator) {
    super.approve(operator, tokenId);
}
```

#### Transfer

onlyAllowedOperatorが追加されている。

Approveと同じくフィルター機能が追加されている。

```solidity
function transferFrom(address from, address to, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperator(from) {
    super.transferFrom(from, to, tokenId);
}

function safeTransferFrom(address from, address to, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperator(from) {
    super.safeTransferFrom(from, to, tokenId);
}

function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data) public override(ERC721, IERC721) onlyAllowedOperator(from) {
    super.safeTransferFrom(from, to, tokenId, data);
}
```

### 全体ソースコード

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "operator-filter-registry/src/DefaultOperatorFilterer.sol";

contract TwitFiNFT is DefaultOperatorFilterer, ERC721Enumerable, Ownable, AccessControl {
	bytes32 public constant MINTER_ROLE = keccak256('MINTER_ROLE');

	string private baseURI;

	struct TokenList {
        address creator;
        uint id;
        uint256 issueDate;
        uint256 modifyDate;
        uint tranferCount;
        string _type;
    }

	event Mint(uint256 tokenId, address _receiver);
	event Burn(uint256 tokenId);
	event TokenTransfer(uint256 tokenId, address oldOwner, address newOwner);

    mapping(uint256 => TokenList) private _listTokens;

    constructor(string memory _name, string memory _symbol, string memory __baseURI) ERC721(_name, _symbol) {
		_grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
        baseURI = __baseURI;
    }

	function _baseURI() internal view override returns (string memory) {
        return baseURI;
    }

    function setBaseURI(string memory newBaseURI) public onlyOwner () {
        baseURI = newBaseURI;
    }

    function setApprovalForAll(address operator, bool approved) public override(ERC721, IERC721) onlyAllowedOperatorApproval(operator) {
        super.setApprovalForAll(operator, approved);
    }

    function approve(address operator, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperatorApproval(operator) {
        super.approve(operator, tokenId);
    }

    function transferFrom(address from, address to, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperator(from) {
        super.transferFrom(from, to, tokenId);
    }

    function safeTransferFrom(address from, address to, uint256 tokenId) public override(ERC721, IERC721) onlyAllowedOperator(from) {
        super.safeTransferFrom(from, to, tokenId);
    }

    function safeTransferFrom(address from, address to, uint256 tokenId, bytes memory data) public override(ERC721, IERC721) onlyAllowedOperator(from) {
        super.safeTransferFrom(from, to, tokenId, data);
    }

	function tokenInfo(uint256 tokenId) public view existingToken(tokenId) returns (TokenList memory) {
        TokenList memory token = _listTokens[tokenId];
        return token;
    }

	function _transfer(address from, address to, uint256 tokenId) internal virtual override {
        super._transfer(from, to, tokenId);

        TokenList memory token = _listTokens[tokenId];

        token.tranferCount += 1;
        token.modifyDate = block.timestamp;

        _listTokens[tokenId] = token;

        emit TokenTransfer(tokenId, from, to);
    }

	function mint(address _receiver, string memory _type, uint256 _tokenId) external onlyMinter {
        _mintToken(_receiver, _type, _tokenId);
    }

	function bulkMint(address[] memory _tos, string[] memory _types, uint256[] memory _tokenIds) external onlyMinter {
        uint8 i;
        for (i = 0; i < _tos.length; i++) {
          _mintToken(_tos[i], _types[i], _tokenIds[i]);
        }
    }

	function _mintToken(address _receiver, string memory _type, uint256 _tokenId) internal returns (uint256) {
		 TokenList memory list = TokenList(
            _receiver,
            _tokenId,
            block.timestamp,
            block.timestamp,
            0,
            _type
        );

        _listTokens[_tokenId] = list;
        _safeMint(_receiver, _tokenId);

        emit Mint(_tokenId, _receiver);
        return _tokenId;
    }

	function burn(uint256 tokenId) public virtual {
        require(_isApprovedOrOwner(_msgSender(), tokenId), "burn caller is not owner nor approved");
        _burn(tokenId);
    }

	function supportsInterface(bytes4 interfaceId)
        public
        view
        virtual
        override(ERC721Enumerable, AccessControl)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }

    function withdraw() public onlyOwner {
        uint amount = address(this).balance;
        require(amount > 0, "Insufficient balance");
        (bool success, ) = payable(owner()).call {
            value: amount
        }("");

        require(success, "Failed to send Matic");
    }

    modifier onlyMinter() {
        require(hasRole(MINTER_ROLE, msg.sender), "Minter: permission denied");
        _;
    }

	modifier existingToken(uint256 _tokenId) {
        require(_exists(_tokenId), "Nonexistent token");
        _;
    }

    receive() payable external {}
}
```

### まとめ

最新バージョンのコンパイラーが使用されていたり、フィルター機能も実装されていたりと、比較的新しい技術が使われている印象でした。

時間がある時に、ERC20のコードも見ていきたいと思います。
