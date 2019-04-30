交易所对接指南
---
这篇文档主要描述了交易所如何一步一步完成和V Systems (VSYS)区块链的对接交互。

For English version Instructions for Exchanges, please click [here](https://vsys.readthedocs.io/en/latest/exchanges.html)

# 准备工作

## 硬件需求

目前阶段，标准硬件配置是2核CPU，16G内存，和512GB高速硬盘的独立主机。

推荐配置是i3 large型号的亚马逊云主机（AWS）

## 软件需求

### 操作系统

可以是所有 Java 1.8 和 Python 可以运行的任何操作系统 (包括 Ubuntu, CentOS, MacOS, Windows 等)。

我们推荐的操作系统是 **Ubuntu 16.04 LTS** (或其更高版本)。

这篇文档我们以Ubuntu 16.04为例进行介绍。

### 软件服务安装

首先我们更新软件包管理器

```shell
$ sudo apt-get update
```

在主机上安装Java 1.8

```shell
$ sudo apt-get install openjdk-8-jdk
```

检查Java版本（需要删除低版本的Java）

```shell
$ java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-0ubuntu0.16.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```

如果您选择编译我们的源码搭建全节点，需要安装Scala编译工具(SBT)

```shell
$ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
$ sudo apt-get update
$ sudo apt-get install sbt
```

如果没有Unzip和Git，请在主机上安装这些软件

```shell
$ sudo apt-get install unzip
$ sudo apt-get install git-core
```

## 启动V全节点

### 第一步: 准备程序

有两种方法准备全节点程序，请任意选择一个方法。

#### 下载编译源代码（方法1）

从GitHub上下载源代码

```shell
$ git clone https://github.com/virtualeconomy/v-systems.git
$ cd vsys
```

用SBT编译源代码。如果您希望编译测试网（Testnet）的V全节点，请运行

```shell
# Compile TestNet V full node
$ sbt -Dnetwork=testnet packageAll
```

如果您希望编译主网（Mainnet）的V全节点，请运行

```shell
# Compile MainNet V full node
$ sbt packageAll
```

编译好的文件将会放在**target/vsys-all-[version].jar**这个位置。复制到你自己的工作目录，例如：

```shell
$ mkdir ../vsys-node
$ cp target/vsys-all-*.jar ../vsys-node/v-systems.jar
$ cd ../vsys-node
```


#### 下载已经编译好的文件（方法2）

如果您不想编译源代码，您也可以选择在 https://github.com/virtualeconomy/v-systems/releases 下载最新的JAR文件。

将**v-systems-[version].jar**保存到您的工作目录。

### 第二步: 配置

设置您的配置文件

```
# V Systems node settings
vsys {
  # Path Settings
  directory = <block data folder path>
  # Application logging level. Could be DEBUG | INFO | WARN | ERROR. Default value is INFO.
  logging-level = INFO
  # P2P Network settings
  network {
    known-peers = ["<peer ip>:<peer port>"]
    black-list-residence-time = 30s
    peers-broadcast-interval = 5s
    connection-timeout = 30s
    # Network address to bind to
    bind-address = "0.0.0.0"
    # Node name to send during handshake. Comment this string out to set random node name.
    # node-name = "My MAINNET node"
    # String with IP address and port to send as external address during handshake. Could be set automatically if uPnP is enabled.
    declared-address = "localhost:9921"
  }
  # Wallet settings
  wallet {
    # Password to protect wallet file
    password = ""
    # Wallet seed as BASE58 string
    # seed = ""
  }
  # Blockchain settings
  blockchain.type = TESTNET   # Should be TESTNET or MAINNET
  # Matcher settings
  matcher.enable = no
  # Minter settings
  miner {
    enable = yes
    offline = no
    quorum = 1
    generation-delay = 1s
    interval-after-last-block-then-generation-is-allowed = 120h
    tf-like-scheduling = no
    # Left to empty as default to minter address
    reward-address = ""
  }
  # Node's REST API settings
  rest-api {
    # Enable/disable node's REST API
    enable = yes
    # Network address to bind to
    bind-address = "0.0.0.0"
    # Hash of API key string
    api-key-hash = "Fo8fR7J1gB3k2WoaE6gYKMwgWfoh9EtZtXAMBxYYCJWG"
  }
  checkpoints.public-key = "A9MX22tXpNdTTx5wBf3CunZz299c1nnca6kH3MzL312L"
}
```
#### 几个比较关键的配置
* **directory**应该设为您自己的工作目录。我们建您挂载一个较大的硬盘，然后工作目录设置到这个硬盘下。

* **known-peers** 这项最好填3个或以上的已知节点。您可以在V explorer查询这些已知节点。现在正在运行的一些节点有：

	```
	# 测试网
	known-peers = ["18.179.34.202:9923", "13.250.53.12:9923", "18.188.219.229:9923"]
	# 主网 (欲知更多节点请和我们联系)
	known-peers = ["13.55.174.115:9921","52.30.23.41:9921","13.113.98.91:9921","3.121.94.10:9921", "54.147.255.148:9921"]
	```

* **blockchain.type** 应该填 TESTNET 或 MAINNET.

* 为安全起见，**api-key-hash**这项最好设置成您自己的哈希值。您可以通过这个命令算出您的api密钥的哈希值：

	```
	curl -X POST -d '<输入任意字符作为您的api密钥>' 'https://test.v.systems/api/utils/hash/secure'
	```

* 最后，我们命名并保存配置文件，例如命名为"vsys.conf"。

### 第3步: 运行

我们建立一个screen并运行

```shell
$ screen -S vsys-node
$ sudo java -jar v-systems*.jar vsys.conf
```

如果需要退出screen，可以在键盘上按 `Ctrl + A + D`。

如果需要再次进入screen看状态，可以运行

```shell
$ screen -x vsys-node
```

## 全节点接口（API）操作

安全提醒：所有的全节点都提供RESTful API进行交互，RESTful API会用到9922端口。安全起见，我们建议交易所修改防火墙规则，**不要将9922端口开放到公网**，仅内网使用。建议开放9921端口(mainnet)和9923端口(testnet)到公网，可以使得节点之间通讯更为发达和通畅。

以下是调用API的几种方法。


### 使用 Python SDK (方法1)

如果您使用python对接，我们强烈建议您使用[pyvsystems](https://github.com/virtualeconomy/pyvsystems)项目完成交互对接。详情请参阅[这里](https://github.com/virtualeconomy/pyvsystems/wiki/PYVSYSTEMS-使用详细指南%28中文%29)。

### 使用 Swagger (方法2)

您可以打开浏览器输入```http://<全节点ip>:9922```使用Swagger，在这里也可以查阅所有可以使用的API。

### 使用 cURL (方法3)

#### 第一步: 准备工作

如果没有安装curl，请安装这个程序，用于发送HTTP请求。

```shell
$ sudo apt-get install curl
```

您可以用以下方法测试连接

```shell
$ curl -X GET --header 'Accept: application/json' 'http://<节点ip>:9922/blocks/height'
```

如果请求成功，您将收到类似这样的回应：

```
{
  "height": 2400326
}
```

#### 第二步: 创建钱包账号

发送HTTP POST请求调用/addresses这个API

```shell
$ curl -X POST --header 'Accept: application/json' --header 'api_key: <节点api密钥>' 'http://<节点 ip>:9922/addresses'
```

如果创建成功，您将会收到一个包含钱包地址的回应：

```
{
  "address": "AUBBmrf5cuBf9XrSaX98mxWmcNBwULqQhQK"
}
```

创建的钱包数据都会保存在```<block data folder path>/wallet/wallet.dat```这个文件里，这个文件最好定期进行备份。

我们也可以根据seed还原钱包，如果查找钱包的seed，可以通过发送HTTP GET调用/addresses/seed/{address} 这个API找到seed（需要带入API密钥），例如

```shell
$ curl -X GET --header 'Accept: application/json' --header 'api_key:  <节点api密钥>' 'http://<节点ip>:9922/addresses/seed/AU9KCwJm6mG9YxSb3LdjVi6LDwRGey1knfy'
```

如果成功将返回类似这样的结果:

```
{
  "address": "AU9KCwJm6mG9YxSb3LdjVi6LDwRGey1knfy",
  "seed": "GY7T8WpppuficZJs9CnEuntLkk4vXw7qkZ1SMtZ3qAas"
}
```

当您知道了seed，您可以通过 [wallet-generator](https://github.com/virtualeconomy/v-wallet-generator) 这个项目来还原钱包地址。

#### 第三步: 钱包地址查询和余额查询

使用 HTTP GET 调用 /addresses API 获取节点内的全部钱包:

```shell
$ curl -X GET 'http://<节点ip>:9922/addresses'
```

如果成功将返回类似结果:

```
[
  "ATy98tPdobDBKA35n5CJed6u3AmxKLT3TTV",
  "ATys7iafCN4xHz9bJyKm4JNfKpk9f1uBBXT",
  "AU1TFgjs3g1NMkXW5CGTGj96t8qijs6ScrP",
  "ATubqbssJKtPfyebkmct4jv9YSue8xrhMLa",
  "AU9KCwJm6mG9YxSb3LdjVi6LDwRGey1knfy",
  "ATwMEAGfNhbRCRSApro8HWG2L65HMMa42KP",
  "AU4rMtj3zEJesVQ94Az8ajncNMgtK6uzeHB",
  "ATtdkLaHDPZQx3LsUfrKisRcrMkvfg18LGa",
  "ATxa6h87rBrNYKDCkagageiPzTMFgcGmefA",
  "AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE"
]
```

随着越来越多的钱包被创建，我们最好限制分批返回结果。我们可以通过 HTTP GET /addresses/seq/{from}/{to} 这个API拿到任意范围的钱包地址，例如

```shell
$ curl -X GET 'http://<节点ip>:9922/addresses/seq/5/10'
```

如果成功将返回类似结果:

```
[
  "ATwMEAGfNhbRCRSApro8HWG2L65HMMa42KP",
  "AU4rMtj3zEJesVQ94Az8ajncNMgtK6uzeHB",
  "ATtdkLaHDPZQx3LsUfrKisRcrMkvfg18LGa",
  "ATxa6h87rBrNYKDCkagageiPzTMFgcGmefA",
  "AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE"
]
```

查询余额可以通过 HTTP GET 调用 /addresses/balance/details/{address}

```shell
$ curl -X GET 'http://<节点ip>:9922/addresses/balance/details/AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE'
```

如果成功将返回类似结果:

```
{
	'address': 'AU8R9ri7eG968zuJuLQVLMiUzRNXvQwNPwE',
	'regular': 109010000000, 		# regular balance
	'available': 108910000000,  		# available balance (regular - leased out)
	'effective': 108910000000,  		# effective balance (regular - leases out + leased in)
	'mintingAverage': 108909964800,  	# for minter used
	'height': 643936
}
```

返回中， ```available``` 的值是最终可用余额 (100000000 = 1 VSYS).

#### 第四步: 创建交易

使用 HTTP POST 调用 /vsys/payment API

```shell
$ curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'api_key: <节点api密钥>' -d '{ \ 
   "amount": 100000000, \ 
   "fee": 10000000, \ 
   "feeScale": 100, \ 
   "sender": "ATtRykARbyJS1RwNsA6Rn1Um3S7FuVSovHK", \ 
   "attachment": "", \ 
   "recipient": "ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX" \ 
 }' 'http://<节点ip>:9922/vsys/payment'
```

在请求的JSON结构中，

```amount``` 是支付给对方的费用，100000000 = 1 VSYS。

```fee``` 是交易费，最小交易费10000000 (0.1 VSYS)。

```feeScale ``` 目前是固定值100。

```sender ``` 是支付方的钱包地址。

```recipient ``` 是接受方的钱包地址。

```attachment ``` 可以说任意字符（最多140个字符），用base58编码填入此项。


如果成功将返回类似结果:

```
{
  "type": 2,
  "id": "EoNQyNouEKg8pDcEEPY2dJL9FMQx61YFk1Sn5EJN8H7K",
  "fee": 10000000,
  "timestamp": 1544083814291691000,
  "proofs": [
    {
      "proofType": "Curve25519",
      "publicKey": "3orvgyRKf45FRyiCkcA3CzAGDvyEpBpXZzYGEGZnpZK5",
      "signature": "t1X2zmw5a2b9iaLgtsHyHKgEmKo6GCFuMFQsZNqj8ZkzpVbRKhWUttqUDfcjzcn5w7VgVVvf8cetr1mh2d2xypQ"
    }
  ],
  "recipient": "ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX",
  "feeScale": 100,
  "amount": 100000000,
  "attachment": ""
}
```

#### 第五步: 查询交易记录

用 HTTP GET 调用 /transactions/address/{address}/limit/{limit} API，例如，

```shell
$ curl -X GET 'http://<节点ip>:9922/transactions/address/ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX/limit/5'
```

如果成功将返回类似结果:

```
[
  [
    {
      "type": 2,
      "id": "EoNQyNouEKg8pDcEEPY2dJL9FMQx61YFk1Sn5EJN8H7K",
      "fee": 10000000,
      "timestamp": 1544083814291691000,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "3orvgyRKf45FRyiCkcA3CzAGDvyEpBpXZzYGEGZnpZK5",
          "signature": "t1X2zmw5a2b9iaLgtsHyHKgEmKo6GCFuMFQsZNqj8ZkzpVbRKhWUttqUDfcjzcn5w7VgVVvf8cetr1mh2d2xypQ"
        }
      ],
      "recipient": "ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX",
      "feeScale": 100,
      "amount": 100000000,
      "attachment": "",
      "status": "Success",
      "feeCharged": 10000000
    },
    {
      "type": 4,
      "id": "FiMiErppddPfFCmehu1ziKNTqyzBFsLRj6gh9y45JKKD",
      "fee": 10000000,
      "timestamp": 1543569020372515800,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
          "signature": "2hpsVXZVs2Wmg5ixD8PqvMJoC3CAqgTqvapYkuFAxbLvoyXRu45q9HXZQyqCzHeiHocGFM8phPkmDuM566Xu59em"
        }
      ],
      "feeScale": 100,
      "leaseId": "D8mGb2YSGyKr5Q3WATnpQP8JvyDdteXwieo5khwsTEyY",
      "status": "Success",
      "feeCharged": 10000000,
      "lease": {
        "type": 3,
        "id": "D8mGb2YSGyKr5Q3WATnpQP8JvyDdteXwieo5khwsTEyY",
        "fee": 10000000,
        "timestamp": 1543569009108564000,
        "proofs": [
          {
            "proofType": "Curve25519",
            "publicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
            "signature": "8TDUgnkNbrPL6VMLFzDnhZvABfRqXitFX46mmvpohsdeRHKaNtWCs5C7m6avaUH2NjiFS7jGFov1CY5s3W8Zc5V"
          }
        ],
        "amount": 100000000,
        "recipient": "AU6GsBinGPqW8zUuvmjgwpBNLfyyTU3p83Q",
        "feeScale": 100
      }
    },
    {
      "type": 3,
      "id": "D8mGb2YSGyKr5Q3WATnpQP8JvyDdteXwieo5khwsTEyY",
      "fee": 10000000,
      "timestamp": 1543569009108564000,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
          "signature": "8TDUgnkNbrPL6VMLFzDnhZvABfRqXitFX46mmvpohsdeRHKaNtWCs5C7m6avaUH2NjiFS7jGFov1CY5s3W8Zc5V"
        }
      ],
      "amount": 100000000,
      "recipient": "AU6GsBinGPqW8zUuvmjgwpBNLfyyTU3p83Q",
      "feeScale": 100,
      "status": "Success",
      "feeCharged": 10000000
    },
    {
      "type": 2,
      "id": "He17g3JXtbXgMiWCTGwnNMPfvfFH5tvyJfYZ7BiWGBZK",
      "fee": 10000000,
      "timestamp": 1543568995612184000,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "CbUPwcCJaMqYSjZGXy4LrkTfV2ncP27Chqyd2QKXfJxn",
          "signature": "24fNpVr8qjrKuDNd8JSeZgkSa4BuS44kxupiAYPHxCbbPMBs2DyT7VDnqAWsJkYZxXWadqiQs7HFxW9uVULrGAtt"
        }
      ],
      "recipient": "ATt6P4vSpBvBTHdV5V9PJEHMFp4msJ1fkkX",
      "feeScale": 100,
      "amount": 100000000,
      "attachment": "",
      "status": "Success",
      "feeCharged": 10000000
    },
    {
      "type": 2,
      "id": "2FVTJUpUJAhZJWkVYHHCG4nRXkYqwQcKsGEK8uwx3A58",
      "fee": 10000000,
      "timestamp": 1543568982176328000,
      "proofs": [
        {
          "proofType": "Curve25519",
          "publicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
          "signature": "3DShPFQLidR1nbTKDrrhpp6SZdiL1hKtjYaGANzGuaWqjpGggPgtrCzw5XYXXktt2sFgWnmFVfTf4gNkmNPNyS2v"
        }
      ],
      "recipient": "AU6GsBinGPqW8zUuvmjgwpBNLfyyTU3p83Q",
      "feeScale": 100,
      "amount": 100000000,
      "attachment": "",
      "status": "Success",
      "feeCharged": 10000000
    }
  ]
]
```

几个常见的交易类型ID:

```
2 = 支付交易
3 = 租赁交易
4 = 取消租赁交易
5 = 挖矿交易
```

### 冷钱包支付签名

为了资产安全，一般交易所会把用户账号上的热钱包的币转移到冷钱包存储，当用户提币的时候再把冷钱包的提出来给用户。冷钱包发起交易的关键技术点在于如何生成签名，生成签名的方法可以参考[pyvsystems](https://github.com/virtualeconomy/pyvsystems)里[account.py](https://github.com/virtualeconomy/pyvsystems/blob/master/account.py)的`send_payment(...)`方法。

当然，您也可以编写自己的程序生成冷钱包签名，我们一步一步解析生成签名的步骤。

例如，我们希望冷钱包发起如下JSON参数的交易：

```
{
  "amount": 1000000000,
  "fee": 10000000,
  "feeScale": 100,
  "timestamp": 1547722056762119200,
  "recipient": "AU6GsBinGPqW8zUuvmjgwpBNLfyyTU3p83Q",
  "senderPublicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
  "attachment": "HXRC"
}
```

把这些参数项按如下顺序转化成Bytes：

```
type_id: 02
timestamp: 15 7a 9d 02 ac 57 d4 00
amount: 00 00 00 00 3b 9a ca 00
tx_fee: 00 00 00 00 00 98 96 80
fee_scale: 00 64
recipient: 05 54 9c 6d f7 b3 76 77 1b 19 ff 3b db 58 d0 4b 49 99 91 66 3c 47 44 4e 42 5f
length of attachment: 00 03
attachment: 31 32 33

```

然后拼接起来：

```
02 15 7a 9d 02 ac 57 d4 00 00 00 00 00 3b 9a ca 00 00 00 00 00 00 98 96 80 00 64 05 54 9c 6d f7 b3 76 77 1b 19 ff 3b db 58 d0 4b 49 99 91 66 3c 47 44 4e 42 5f 00 03 31 32 33
```

最终，我们用冷钱包私钥通过[curve25519](https://github.com/tgalal/python-axolotl-curve25519)的ed25519方法来完成签名。

```
(For reference only. The signature will be different if generate again)
72 74 61 73 6d 50 31 4c 63 48 79 5a 63 71 35 36 67 52 34 78 57 45 35 78 68 54 78 59 35 33 6f 6f 6f 4d 32 53 61 36 63 75 42 52 61 78 72 71 33 39 63 54 56 6b 39 4d 67 4d 76 6e 38 61 5a 45 6d 4e 78 6b 56 39 55 39 63 62 41 6a 43 50 4d 68 48 46 6f 51 33 57 69 66 57
```

把得到的签名用base58编码，然后放到JSON的`signature`项。

```
{
  "amount": 1000000000,
  "fee": 10000000,
  "feeScale": 100,
  "timestamp": 1547722056762119200,
  "recipient": "AU6GsBinGPqW8zUuvmjgwpBNLfyyTU3p83Q",
  "senderPublicKey": "B2Khd89jtnpuzGdnyGRcnKycZMBCo6PsotFcWWi1wMDV",
  "attachment": "HXRC",
  "signature": "rtasmP1LcHyZcq56gR4xWE5xhTxY53oooM2Sa6cuBRaxrq39cTVk9MgMvn8aZEmNxkV9U9cbAjCPMhHFoQ3WifW"
}
```

将这个JSON传递给全节点，然后全节点用API`/vsys/broadcast/payment`广播到网络上。

## 常见问题
* [Exchange Integration FAQ](https://vsys.readthedocs.io/en/latest/FAQ.html)

## 密钥及钱包生成工具

* [Wallet Generator](https://github.com/virtualeconomy/v-wallet-generator) (Scala)
* [VSYS HDkey](https://github.com/virtualeconomy/VSYS_HDkey_java) (Java)
* [VSYS HDkey](https://github.com/virtualeconomy/VSYS_HDkey_go) (Go)
