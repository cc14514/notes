# hardhat + vscode 写的 NFT


## 创建一个新的目录，进入该目录并初始化 npm：

```
mkdir my-nft && cd my-nft
npm init -y
```

## 安装 Hardhat：

```
npm install --save-dev hardhat
```

## 初始化 Hardhat 项目：

```
选择 "Create a sample project"，选择 "JavaScript"，然后选择 "Yes" 以安装依赖。
```

## 安装 OpenZeppelin 的 ERC721 库：

```
npm install --save-dev @openzeppelin/contracts@3.4.0
```

## 创建 NFT 合约：

在 contracts/ 目录下创建一个新的 Solidity 文件，如 MyNFT.sol，并在其中编写以下代码：

```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "MNFT") {}

    function mint(address to, uint256 tokenId) public {
        _mint(to, tokenId);
    }
}
```

该合约继承了 OpenZeppelin 的 ERC721 合约，并在构造函数中设置了代币名称和代币符号。还有一个名为 mint 的公共函数，用于铸造新的 NFT。

## 部署合约：

在 scripts/ 目录下创建一个新的 JavaScript 文件，如 deploy.js，并在其中编写以下代码：

```javascript
const { ethers } = require("hardhat");

async function main() {
  const MyNFT = await ethers.getContractFactory("MyNFT");
  const myNFT = await MyNFT.deploy();

  console.log("MyNFT deployed to:", myNFT.address);
}

main().then(() => process.exit(0)).catch(error => {
  console.error(error);
  process.exit(1);
});
```

该脚本使用 Hardhat 的 ethers 库获取合约工厂，然后部署新的 MyNFT 合约。
在终端中运行 npx hardhat run scripts/deploy.js 以部署合约。

## 铸造 NFT：

在 scripts/ 目录下创建一个新的 JavaScript 文件，如 mint.js，并在其中编写以下代码：

```javascript
const { ethers } = require("hardhat");

async function main() {
  const MyNFT = await ethers.getContractFactory("MyNFT");
  const myNFT = await MyNFT.attach("<MyNFT contract address>");

  await myNFT.mint("<recipient address>", 1);

  console.log("NFT minted!");
}

main().then(() => process.exit(0)).catch(error => {
  console.error(error);
  process.exit(1);
});
```

将 <MyNFT contract address> 替换为您的合约地址，并将 <recipient address> 替换为您要铸造 NFT 的地址。
在终端中运行 npx hardhat run scripts/mint.js 以铸造 NFT。
