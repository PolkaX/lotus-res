
## 环境依赖
- go (1.13 or higher)
- gcc (7.4.0 or higher)
- git (version 2 or higher)
- bzr (some go dependency needs this)
- jq
- pkg-config
- opencl-icd-loader
- opencl driver (like nvidia-opencl on arch) (for GPU acceleration) 
- opencl-headers (build)
- rustup (proofs build)
- llvm (proofs build)
- clang (proofs build)

### 安装各种依赖
Ubuntu (build):
```sh
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt update
sudo apt install golang-go gcc git bzr jq pkg-config mesa-opencl-icd ocl-icd-opencl-dev
```
## 编译
lotus目前暂未发布二进制，所以需要自行编译二进制
```
$ git clone https://github.com/filecoin-project/lotus.git
$ cd lotus/
```

```
$ make clean all
$ sudo make install
```
## 启动节点
当前 Lotus 构建版本默认使用 `build` 目录中的创世区块和 引导文件自动加入 Lotus 开发网络。


```sh
$ lotus daemon
```
查看当前连接的节点数：
```sh
$ lotus net peers | wc -l
```
查看当前节点同步高度：
```sh
$ lotus sync wait
```
创建地址：
```sh
$ lotus wallet new
t3...
```
可以去 [dashboard](https://lotus-metrics.kittyhawk.wtf/chain) 查看当前开发网络最新区块高度和其他网络指标。

## 获取测试代币，创建矿工
目前官方提供了一个[水龙头](https://lotus-faucet.kittyhawk.wtf/),Lotus 获取开发网代币有两种方式，一种是 [Send Funds] 这个接口每次请求会给你的钱包地址充值 0.0000000005 个测试代币。并且为了防止 薅羊毛，接口请求的次数是有限制的。每个 IP 每 5 分钟发出一次请求，初始并发次数为5，最大并发数为 20。具体详细的限制请参考： https://github.com/filecoin-project/lotus/pull/384

下面我们来看下当前网络创建一个矿工需要抵押多少代币：
```sh
# lotus state pledge-collateral
10.276412838102150792
```
目前创建矿工手续费大约10，也就是说，这个接口无法满足创建一个矿工。
所以 Lotus 还提供了另外一种申请测试代币的方式：[Create Miner]，就是直接申请一笔足够创建矿工的资金，同时帮你创建一个矿工。 这个接口一个地址只能调用一次，每次调用费时大概 5 分钟，一般我们都是直接用这个接口去申请代币。

## 初始化矿工，开始挖矿
创建矿工成功之后，Faucet 页面会返回类似下面的指令。
```
[CREATING STORAGE MINER]
Gas Funds:   bafy2bzacea27myklbxt6ly5m426a6tvrnrt4bgahg45epjfplbwfxqagt5mqu - OK
Miner Actor: bafy2bzacecq2p3sfrlrpjlw3r2pggi3dstsqxot53atyejir4je6qx3ecuuaq - OK
New storage miners address is: t0571
To initialize the storage miner run the following command:
lotus-storage-miner init --actor=t0571 --owner=t3uzr65jmr2hwzr7uu2irf6uclrqytmw2utmn4hyop56yqmci2pxejitpoxara2kdsfvsraxdcjgfdopfbhs5q
```
这条命令把矿工相应的信息提交到区块链，如果上链成功的话，此命令应成功返回。
下面是执行过程，主要是获取用于生成 PoSts的参数到本地(/var/tmp/filecoin-proof-parameters)，这个过程根据网络情况可能花费若干小时。
**如果这一步不成功，翻墙试试**
```
./lotus-storage-miner init --actor=t0571 --owner=t3uzr65jmr2hwzr7uu2irf6uclrqytmw2utmn4hyop56yqmci2pxejitpoxara2kdsfvsraxdcjgfdopfbhs5q
2019-11-14T15:53:08.952+0800	INFO	main	lotus-storage-miner/init.go:59	Initializing lotus storage miner
2019-11-14T15:53:08.952+0800	INFO	main	lotus-storage-miner/init.go:61	Checking proof parameters
2019-11-14T15:53:08.954+0800	INFO	build	build/paramfetch.go:113	Parameter file /var/tmp/filecoin-proof-parameters/v15-proof-of-spacetime-rational-535d1050e3adca2a0dfe6c3c0c4fa12097c9a7835fb969042f82a507b13310e0.vk is ok
2019-11-14T16:08:41.054+0800	INFO	build	build/paramfetch.go:113	Parameter file /var/tmp/filecoin-proof-parameters/v15-stacked-proof-of-replication-967b11bb59be11b7dc6f2b627520ba450a3aa50846dbbf886cb8b735fe25c4e7.vk is ok
2019-11-14T16:08:41.054+0800	INFO	build	build/paramfetch.go:113	Parameter file /var/tmp/filecoin-proof-parameters/v15-stacked-proof-of-replication-d01cd22091627b721c60a3375b5219af653fb9f6928c70aa7400587d396bc07a.vk is ok
 391.00 MiB / 391.00 MiB [========================================================================] 100.00% 2.12 MiB/s 3m4s
2019-11-14T16:11:48.278+0800	INFO	build	build/paramfetch.go:130	Fetching /var/tmp/filecoin-proof-parameters/v15-stacked-proof-of-replication-0c0b444c6f31d11c8e98003cc99a3b938db26b77a296d4253cda8945c234266d.params from https://ipfs.io/ipfs/
 776.12 MiB / 3.30 GiB [================>-------------------------------------------------------]  22.95% 1.73 MiB/s 25m03s
 2019-11-15T10:13:06.295+0800	INFO	main	lotus-storage-miner/init.go:66	Checking if repo exists
2019-11-15T10:13:06.317+0800	INFO	main	lotus-storage-miner/init.go:82	Trying to connect to full node RPC
2019-11-15T10:13:06.811+0800	INFO	main	lotus-storage-miner/init.go:91	Checking full node version
2019-11-15T10:13:06.918+0800	INFO	main	lotus-storage-miner/init.go:102	Initializing repo
2019-11-15T10:13:06.919+0800	INFO	repo	repo/fsrepo.go:98	Initializing repo at '/home/fy/.lotusstorage'
2019-11-15T10:13:07.079+0800	INFO	main	lotus-storage-miner/init.go:131	Initializing libp2p identity

...

```
我在这一步等待了很久无返回，Ctrl+C也无法结束进程，等待这一步成功之后

运行矿工，开始挖矿：
```sh
lotus-storage-miner run
```
你可以通过下面的方式查看矿工信息：
```sh
lotus-storage-miner info
```
## 存储挖矿
## 检索挖矿

在lotus节点搭建过程中，根据官方推荐顺便搭建了一个监控台，可以实时监控展示节点与链的运行状态。

## 环境依赖
- go (1.13 or higher)
- docker
- docker-compose
## 准备工作
### grafana

[grafana](https://grafana.com/docs/installation/debian/): 用来实时展示状态，可以根据需求自由定制

### influxDB

[influxDB](https://www.influxdata.com/products/influxdb-overview/): 获取节点数据，为监控台提供数据源
### docker-compose用法
Docker Compose 是一个用来定义和运行复杂应用的 Docker 工具。
使用 Docker Compose 不再需要使用 shell 脚本来启动容器。(通过 docker-compose.yml 配置)

安装：
```
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
卸载：
```
sudo rm /usr/local/bin/docker-compose
```
基础命令：
*需要在 docker-compose.yml 所在文件夹中执行命令*

停止现有 docker-compose 中的容器：
```
docker-compose down
```
重新拉取镜像：
```
docker-compose pull
```
后台启动 docker-compose 中的容器：
```
docker-compose up -d
```
### Stats
stats是lotus源码中提供的一个工具，它的作用是将lotus节点数据推送给influxdb，我们需要先编译，接下来的操作中会用到。

源码目录是：
lotus/tools/stats

编译方法:
```
go build -o stats *.go
```
在`lotus/tools/stats`下有四个重要的文件
- chain.dashboard.json lotus提供的dashboard模板，可以根据需要个性化设置
- docker-compose.yml docker-compose配置文件，里边的name，port端口可以自己修改，但是对应的env.stats文件内的端口也需要修改。
    - 8083：访问web页面的地址，8083为默认端口；
    - 8086：数据写入influxdb的地址，8086为默认端口；
- env.stats 环境配置文件，与docker-compose.yml对应
- setup.bash grafana初始化配置脚本

**服务器上部署时安全组策略记得开启准入访问的端口，以免无法访问！**

## 开始
### 步骤1
在lotus/tools/stats下启动grafana和influxdb，将dashboard导入grafana，记下返回的url，这个url就是访问dashboard的链接。
```
docker-compose up -d #后台启动grafana,influxdb
./setup.bash    #导入dashboard模板
```
### 步骤2
```
. env.stats && ./stats  #将数据写入influxdb
```
这一步执行完之后就需要漫长的等待，等待stats将数据写入influxdb，具体时间根据服务器性能和当前lotus链块高决定，块高越大时间越久。等追上最新块高，grafana就会显示监控数据。grafana默认登录账户名和密码都是 **admin**

