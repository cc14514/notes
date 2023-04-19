# 以太坊 geth 本地测试节点

开发和测试 dapp 的合约有很多现成的节点方案比如 `hardhat`，没有好坏之分，只是习惯不同。本人习惯自己搭建，那么开发和调试合约就必须要有一个本地的低能耗节点了，以太坊的 `clique` 共识模块提供的 `POA` 共识很符合开发需求，言归正传直接上攻略

## 下载并编译 geth

省略安装配置 `golang` 编译环境，如果不喜欢编译，其实可以跳过这一步直接下载编译好的 `bin` 

```bash
git clone git@github.com:ethereum/go-ethereum.git
cd go-ethereum
make geth

build/bin/geth --help
```

编译成功后建议将 build/bin/geth 链接到 PATH 对应的目录中，例如 /usr/local/bin 等

## 制作 genesis.json

### 1、创建账户


```bash
geth --datadir ./data account new
echo 1 > pwd.txt
```    

此处假设账户 `b0ab1f54bd403f8cd360b39c3c45ee2055a0aaa0`, 密码 `1`


### 2、制作 genesis.json

手动创建即可，复制内容如下并保存为 `genesis.json`：

```json
{
  "config": {
    "chainId": 12345,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "berlinBlock": 0,
    "clique": {
      "period": 5,
      "epoch": 30000
    }
  },
  "difficulty": "1",
  "gasLimit": "8000000",
  "extradata": "0x0000000000000000000000000000000000000000000000000000000000000000b0ab1f54bd403f8cd360b39c3c45ee2055a0aaa00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "alloc": {
    "b0ab1f54bd403f8cd360b39c3c45ee2055a0aaa0": { "balance": "999999999000000000000000000" }
  }
}
```

需要注意的是 `clique` 共识的第一个签名出块节点是在 `extradata` 中指定的，替换其中的签名节点为你自己的 `account` 地址


### 3、初始化节点

```bash
geth --datadir ./data init ./genesis.json
```

在当前目录中初始化创世区块

### 4、启动节点并出块

```bash
geth --datadir=./data --networkid 12345 --mine --miner.etherbase=b0ab1f54bd403f8cd360b39c3c45ee2055a0aaa0 --nodiscover --unlock b0ab1f54bd403f8cd360b39c3c45ee2055a0aaa0 --password $./pwd.txt console
```

进入控制台后可以看到已经在按照配置 `5` 秒一个块了


## 资料

https://geth.ethereum.org/docs/fundamentals/private-network