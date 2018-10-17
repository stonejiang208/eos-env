# 搭建EOS开发环境实作

给小伙伴分享一篇搭建EOS开发环境的入门篇。看完这一篇，期望你也能

1. 尽可能最快搭建开发环境，少掉进坑里，本文尽可能把已知的坑标注一下
2. 会搭建测试的节点
3. 了解智能合约，有哪些预备环境
4. “开发”，部署和验证智能合约-发币、转帐、查余额
5. 了解后续的道路


## 准备环境
 
根据我的经验，先理解一个整体，这样每一步自己在干什么才清楚。下面先奉上
EOS的架构图。

https://files.readme.io/582e059-411_DevRelations_NodeosGraphic_Option3.png

### 预备知识
  
   如果对Docker不熟悉，先停下来，花点时间补一下，开发EOS对Docker要求不高，简单
   看一下应该就可以了。
   

### 操作系统
    开发EOS最佳的环境是Linux (Ubuntu 18.04)，如果不是，用Docker也一样。

### 硬件
    强烈建议创建一台云主机，最好是海外的，内存最好是16G的，按时间收费，成本其实很低。 
    推荐使用Digital,点这里咱俩都可以获得优惠 (各$25)。
    https://m.do.co/c/f989f443a0d3
    
    
    如果在本地，最好要有良好的网络环境，内存最好不要少于8G。
    
### 相关基础
  
     本文会涉及到以下知识点:
     C++
     git
     cmake
     ubuntu 以及Bash基础
     Docker 基础
     
### 官方文档

https://developers.eos.io/eosio-home/docs/introduction

## 搭建测试用的节点

分两步

1. 先创建一个目录，我这里在是根目录上建的

2. 拉取Docker镜像并启动eosio的节点

```
mkdir -p /contracts
mkdir -p /mylocal

```
这个目录下一步会用到，让容器挂接主机的这个目录。


``` bash
   docker run --name eosio \
  --publish 8888:8888 \
  --publish 127.0.0.1:5555:5555 \
  -v /contracts:contracts \
  -v /mylocal:/usr/local \
  -w /contracts \
  -d \
  eosio/eos \
  /bin/bash -c \
  "keosd --http-server-address=0.0.0.0:5555 & exec nodeos -e -p eosio --plugin eosio::producer_plugin --plugin eosio::history_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:7777 --access-control-allow-origin=* --contracts-console --http-validate-host=false --filter-on='*'"
```

官方文档使用的端口是7777，在实际操作中，发现配置文件中配置的是8888。稍微注意一点是你的主机中其它程序或容器没有使用8888端口。

用下面的方法检查容器是否成功。

```bash
docker logs --tail 10 eosio

```
如果看到类似下面的输出，你就成有一个完美的开始。

```bash
1929001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366974ce4e2a... #13929 @ 2018-05-23T16:32:09.000 signed by eosio [trxs: 0, lib: 13928, confirmed: 0]
1929502ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366aea085023... #13930 @ 2018-05-23T16:32:09.500 signed by eosio [trxs: 0, lib: 13929, confirmed: 0]
1930002ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366b7f074fdd... #13931 @ 2018-05-23T16:32:10.000 signed by eosio [trxs: 0, lib: 13930, confirmed: 0]
1930501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366cd8222adb... #13932 @ 2018-05-23T16:32:10.500 signed by eosio [trxs: 0, lib: 13931, confirmed: 0]
1931002ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366d5c1ec38d... #13933 @ 2018-05-23T16:32:11.000 signed by eosio [trxs: 0, lib: 13932, confirmed: 0]
1931501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366e45c1f235... #13934 @ 2018-05-23T16:32:11.500 signed by eosio [trxs: 0, lib: 13933, confirmed: 0]
1932001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366f98adb324... #13935 @ 2018-05-23T16:32:12.000 signed by eosio [trxs: 0, lib: 13934, confirmed: 0]
1932501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00003670a0f01daa... #13936 @ 2018-05-23T16:32:12.500 signed by eosio [trxs: 0, lib: 13935, confirmed: 0]
1933001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00003671e8b36e1e... #13937 @ 2018-05-23T16:32:13.000 signed by eosio [trxs: 0, lib: 13936, confirmed: 0]
1933501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000367257fe1623... #13938 @ 2018-05-23T16:32:13.500 signed by eosio [trxs: 0, lib: 13937, confirmed: 0]

```

更进一步，可以检查一下钱包，检查一下结点

#### 检查钱包


### 检查结点


### Docker使用的小福利

#### 启/停容器

#### 进入容器

#### 删除容器


## 编译CDT (智能合约开发工具链)

进入到eosio容器中，后面的工作，我们都在容器中进行。步骤在如下

1. 给容器安装sudo, git

```bash
apt-get update
apt-get install -y git sudo
```

2. 抓取CDT的1.2.1版本  （版本是一个坑，写本文的时候，一定要1.2.1）

```bash
cd /contracts

git clone --recursive https://github.com/eosio/eosio.cdt --branch v1.2.1 --single-branch
cd eosio.cdt
```


3. 编译

```bash
./build.sh SYS
```

build.sh要带一个参数SYS，这个后面我们要用得到。


4. 安装

```bash

sudo ./install.sh

```

## 理解一下智能合约

// 

## 开发部署智能合约前的准备

开发部署智能合约要先准备钱包，准备帐号。本节操作跟官网一致 （先要撸代码稍后补充文档)


## 开发和部署智能合约

## 编译调试 Hello World让Bob和Alice约会

本节操作跟官网一致

另外找时间来补充完整

## 编译调试token:发币转帐查额

本节操作跟官网一致

另外找时间来补充完整
