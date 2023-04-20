# ERC721 标准

ERC721 是非同质化代币（NFT）标准。ERC721 通过在以太坊区块链上创建代币，使得每个代币都是唯一的、不可分割和可跟踪的。

## 简介

ERC721 标准实现了以太坊上的非同质化代币。每个 ERC721 代币都是唯一的，不可分割和可跟踪的，这意味着每个 ERC721 代币都具有独特的属性和价值。ERC721 代币的价值通常是由它的稀缺性、独特性和可交易性所决定的。

## ERC721 合约

ERC721 合约是实现 ERC721 标准的智能合约。通过使用 ERC721 合约，用户可以创建、铸造、销毁和转移 ERC721 代币。ERC721 合约必须实现一组标准的接口，以便其他合约和应用程序可以与之交互。这些标准接口确保了 ERC721 合约可以与其他 ERC721 合约和应用程序无缝协作。

# 标准接口

以下是 ERC721 标准定义的接口：


## ERC721 接口

```javascript
interface ERC721 /* is ERC165 */ {
    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
    event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);
    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);

    function balanceOf(address _owner) external view returns (uint256);
    function ownerOf(uint256 _tokenId) external view returns (address);
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;
    function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
    function approve(address _approved, uint256 _tokenId) external payable;
    function setApprovalForAll(address _operator, bool _approved) external;
    function getApproved(uint256 _tokenId) external view returns (address);
    function isApprovedForAll(address _owner, address _operator) external view returns (bool);
}
```

### balanceOf

```javascript
function balanceOf(address owner) external view returns (uint256);

该函数接受一个地址参数 owner，返回该地址所拥有的 ERC721 代币数量。
```

### ownerOf

```javascript
function ownerOf(uint256 tokenId) external view returns (address);

该函数接受一个整数参数 tokenId，返回该代币的拥有者地址。
```

### safeTransferFrom

```javascript
function safeTransferFrom(address from, address to, uint256 tokenId, bytes calldata data) external;

该函数将一个 ERC721 代币从一个地址转移到另一个地址，并调用目标地址的接收函数，确保转移的安全性。其中参数 from 是当前拥有者地址，参数 to 是新的拥有者地址，参数 tokenId 是要转移的代币 ID，参数 data 是可选的数据参数。
```

### safeTransferFrom

```javascript
function safeTransferFrom(address from, address to, uint256 tokenId) external;

与上一个函数类似，该函数也将一个 ERC721 代币从一个地址转移到另一个地址，但不会携带额外的数据参数。
```

### transferFrom

```javascript
function transferFrom(address from, address to, uint256 tokenId) external;

该函数将一个 ERC721 代币从一个地址转移到另一个地址，其中参数 from 是当前拥有者地址，参数 to 是新的拥有者地址，参数 tokenId 是要转移的代币 ID。
```

### approve

```javascript
function approve(address to, uint256 tokenId) external;

该函数将一个 ERC721 代币的授权转移到另一个地址，其中参数 to 是新的授权地址，参数 tokenId 是要授权的代币 ID。
```

### setApprovalForAll

```javascript
function setApprovalForAll(address operator, bool approved) external;

该函数将当前地址对所有 ERC721 代币的授权转移到另一个地址，其中参数 operator 是新的授权地址，参数 approved 表示是否授权。
```

### getApproved

```javascript
function getApproved(uint256 tokenId) external view returns (address);

该函数返回一个 ERC721 代币当前的授权地址，其中参数 tokenId 是要查询的代币 ID。
```

### isApprovedForAll

```javascript
function isApprovedForAll(address owner, address operator) external view returns (bool);

该函数返回一个地址是否已被授权管理一个地址所拥有的所有 ERC721 代币，其中参数 owner 是代币拥有者地址，参数 operator 是被查询的
```



## ERC721Metadata 接口

继承自 ERC721，它定义了一些关于 NFT 元数据的标准方法。

```javascript
interface ERC721Metadata /* is ERC721 */ {
    function name() external view returns (string memory _name);
    function symbol() external view returns (string memory _symbol);
    function tokenURI(uint256 _tokenId) external view returns (string memory);
}
```

>function name() external view returns (string memory) 返回 NFT 的名称。

>function symbol() external view returns (string memory) 返回 NFT 的简称。

> function tokenURI(uint256 tokenId) external view returns (string memory) 返回与指定 NFT 关联的元数据 URI，该 URI 包含有关 NFT 的详细信息，如名称、描述、图像等。

这些方法在实现 NFT 元数据时非常有用，因为它们允许开发者标准化他们的元数据格式，并为用户提供更多信息和交互性。

## ERC721Enumerable 接口

继承了 ERC721 和 ERC165 接口，并新增了以下方法：

```javascript
interface ERC721Enumerable /* is ERC721 */ {
    function totalSupply() external view returns (uint256);
    function tokenOfOwnerByIndex(address _owner, uint256 _index) external view returns (uint256 _tokenId);
    function tokenByIndex(uint256 _index) external view returns (uint256);
}
```


>totalSupply(): 返回已经铸造的 NFT 的数量。

>tokenByIndex(uint256 index): 返回指定索引处的 NFT ID。在所有铸造的 NFT 中，它定义了一个唯一的顺序。

>tokenOfOwnerByIndex(address owner, uint256 index): 返回指定地址的 NFT 列表中指定索引处的 NFT ID。在所有该地址拥有的 NFT 中，它定义了一个唯一的顺序。

这些方法使得 ERC721 合约支持按照顺序枚举所有已铸造的 NFT，或者按照拥有者地址枚举特定地址拥有的 NFT。这对于一些场景，例如 NFT 游戏，非常有用。


# 官方资料

* https://ethereum.org/en/developers/docs/standards/tokens/erc-721/
* https://eips.ethereum.org/EIPS/eip-721

