# ironfish-zh-docs

## 1、Testnet Phase 2

- [ironfish官网](https://ironfish.network/)
  
- [活动内容](https://testnet.ironfish.network/about)
  
- [排行榜](https://testnet.ironfish.network/leaderboard)
  
- [ironfish区块链浏览器](https://explorer.ironfish.network/)
  
- [安装指南](https://ironfish.network/docs/onboarding/iron-fish-tutorial)
  

### 2、矿池

- 官方矿池
  
  - pool.ironfish.network
- peak-pool.com提供的矿池
  
  - stratum://peak-pool.com:9034
- discord用户AX、#2649提供的矿池
  
  - 161.97.131.247

## 3、安装教程

> 环境: Ubuntu20.04 docker
> 
> 功能:
> 
> node同步区块，用于转账，
> 
> miner提供算力，连接官方矿池或者自己的node

#### 3.1 拉取镜像(使用梯子或者从其他服务器导入)

```shell
docker pull ghcr.io/iron-fish/ironfish:latest
```

#### 3.2 启动node

```shell
docker run -itd --name node --net host --volume /root/.node:/root/.ironfish ghcr.io/iron-fish/ironfish:latest start --rpc.tcp --rpc.tcp.host=0.0.0.0 --rpc.tcp.port=8001
# 根据自己情况设置参数
# --name node 设置容器名称为 node
# --net host 设置网络模式为host
# --volume /root/.node:/root/.ironfish  将主机/root/.node映射到容器/root/.ironfish位置
# --rpc.tcp --rpc.tcp.host=0.0.0.0 --rpc.tcp.port=8001 启动node时，启动ironfish tcp服务，端口为8001
```

#### 3.3 设置node

设置涂鸦

```shell
docker exec -it node bash -c "ironfish config:set blockGraffiti 涂鸦名称"
```

启动node会默认创建钱包，以下是对钱包的操作命令

```shell
# 查看钱包
docker exec -it node bash -c "ironfish accounts:list"
# 创建新钱包
docker exec -it node bash -c "ironfish accounts:create"
# 设置为默认钱包
docker exec -it node bash -c "ironfish accounts:use 钱包名称"
# 导出钱包密钥
docker exec -it node bash -c "ironfish accounts:export 钱包名称"
# 导入钱包密钥
docker exec -it node bash -c "ironfish accounts:import"
```

**使用ironfish deposit转账获得积分 (每次0.1个iron，获得1积分)**

```shell
docker exec -it node bash -c "ironfish deposit --confirm"
```

**托管节点获得积分 (运行12小时 10积分)**

```shell
docker exec -it node bash -c "ironfish testnet"

# 执行查看以下命令Telemetry会显示为STARTED
docker exec -it node bash -c "ironfish status"

Telemetry            STARTED - 3150 <- 0 pending
```

#### 3.4 启动miner(启动miner时设置CPU数并非越多越好，需自己测试)

加入自己的node

```shell
docker run -itd --name miner --net host --cpus 24 --volume /root/.miner:/root/.ironfish ghcr.io/iron-fish/ironfish:latest miners:start --threads=24 --rpc.tcp --rpc.tcp.host=<node的ip地址> --rpc.tcp.port=8001
# 根据自己的情况设置参数
# --cpus 24 设置cpu限制为24线程
# miners:start --threads=24 启动miner时使用24线程
# --rpc.tcp --rpc.tcp.host=<node的ip地址> --rpc.tcp.port=8001 启动miner时,使用node设置的的ip和端口
```

加入官方矿池

```shell
docker run -itd --name miner --net host --cpus 24 --volume /root/.miner:/root/.ironfish ghcr.io/iron-fish/ironfish:latest miners:start --threads=24 --pool pool.ironfish.network --address <自己的公钥>
# --pool pool.ironfish.network 加入官方矿池
# --address <自己的公钥> 设置自己用于挖矿的公钥
```

#### 3.5 更新ironfish

> 因为启动node时指定了 -v参数进行了文件映射，所以即使容器了容器，还会保留链数据，只需要接着同步就行

```shell
# 拉取新的ironfish镜像
docker pull ghcr.io/iron-fish/ironfish:latest
# 删除旧版本ironfish启动的node
docker rm -f node
# 启动新版本node (参照3.2)
docker run -itd --name node --net host --volume /root/.node:/root/.ironfish ghcr.io/iron-fish/ironfish:latest start --rpc.tcp --rpc.tcp.host=0.0.0.0 --rpc.tcp.port=8001
```

### 4、目前bug

##### 4.1 显示代币信息异常、转账一直报错

1、之前有一笔转账未完成，账户资金未刷新

```shell
# 执行以下命令，balance和available不一致
docker exec -it node /bin/bash -c "ironfish accounts:balance"

The balance is: $IRON 6.53009896 ($ORE 653,009,896)
Amount available to spend: $IRON 0.03009896 ($ORE 3,009,896)
```

2、账户信息错误 (个人猜测)

```shell
# 执行可能需要一段时间
docker exec -it node /bin/bash -c "ironfish accounts:rescan"
```
