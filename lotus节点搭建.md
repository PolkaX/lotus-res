
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
:~/work/code/lotus$ ./lotus-storage-miner init --actor=t02190 --owner=t3sd74srma7kxlhexhfboawuxrgfxzs66ti3kre3khldubde5q2odgn35omveo5qxtry4qmly7o72dcwjhkxya
2019-12-11T10:43:44.326+0800	INFO	main	lotus-storage-miner/init.go:80	Initializing lotus storage miner
2019-12-11T10:43:44.336+0800	INFO	main	lotus-storage-miner/init.go:84	Checking proof parameters
2019-12-11T10:43:44.339+0800	INFO	build	build/paramfetch.go:121	Parameter file /var/tmp/filecoin-proof-parameters/v20-proof-of-spacetime-election-512f5e6dc00a37fa13c8b0e468188f85957b7bf1ab36d17fb9fe9ed49ae8d657.vk is ok
 734.39 MiB / 734.39 MiB [=====================================================================================================================] 100.00% 4.14 MiB/s 2m57s
2019-12-11T10:46:43.476+0800	INFO	build	build/paramfetch.go:121	Parameter file /var/tmp/filecoin-proof-parameters/v20-proof-of-spacetime-election-6c7cbfe7eed40b6c0b23a213a70648770aed65d9ca03ae85451573c18532304b.params is ok
2019-12-11T10:46:43.476+0800	INFO	build	build/paramfetch.go:138	Fetching /var/tmp/filecoin-proof-parameters/v20-stacked-proof-of-replication-f571ee2386f4c65a68e802747f2d78691006fc81a67971c4d9641403fffece16.params from https://ipfs.io/ipfs/
2019-12-11T10:46:43.476+0800	INFO	build	build/paramfetch.go:156	GET https://ipfs.io/ipfs/QmSAHu14Pe8iav6BYCt9XkpHJ73XM7tcpY4d9JK9BST9HU
 4.87 GiB / 4.87 GiB [========================================================================================================================] 100.00% 4.93 MiB/s 16m50s
2019-12-11T11:03:46.497+0800	INFO	build	build/paramfetch.go:121	Parameter file /var/tmp/filecoin-proof-parameters/v20-stacked-proof-of-replication-f571ee2386f4c65a68e802747f2d78691006fc81a67971c4d9641403fffece16.params is ok
2019-12-11T11:03:46.497+0800	INFO	main	lotus-storage-miner/init.go:89	Trying to connect to full node RPC
2019-12-11T11:03:46.610+0800	INFO	main	lotus-storage-miner/init.go:98	Checking full node sync status
Worker 0: Target: [bafy2bzacecruhzwqyz7ics4rowftdfqps6z4xnvjopc45fkxsm56mxstamqe2]      State: message sync     Height: 0
Done!
2019-12-11T11:03:46.809+0800	INFO	main	lotus-storage-miner/init.go:106	Checking if repo exists
2019-12-11T11:03:46.822+0800	INFO	main	lotus-storage-miner/init.go:122	Checking full node version
2019-12-11T11:03:46.837+0800	INFO	main	lotus-storage-miner/init.go:133	Initializing repo
2019-12-11T11:03:46.837+0800	INFO	repo	repo/fsrepo.go:97	Initializing repo at '/home/fy/.lotusstorage'
2019-12-11T11:03:47.038+0800	INFO	main	lotus-storage-miner/init.go:330	Initializing libp2p identity
2019-12-11T11:03:48.140+0800	INFO	badger	badger@v1.6.0-rc1/logger.go:46	All 0 tables opened in 0s

2019-12-11T11:03:48.543+0800	INFO	main	lotus-storage-miner/init.go:490	Waiting for message: bafy2bzacebt7fprsbkxbqynloklsowmsoezijvxc3l2z57wz7t5rjzalmjvyi
2019-12-11T11:05:04.931+0800	INFO	main	lotus-storage-miner/init.go:416	Created new storage miner: t02190
2019-12-11T11:05:05.977+0800	INFO	main	lotus-storage-miner/init.go:206	Storage miner successfully created, you can now start it with 'lotus-storage-miner run'

```
在这一步等待很久，等待这一步成功之后

运行矿工，开始挖矿：
```sh
# lotus-storage-miner run
019-12-11T11:05:48.275+0800	INFO	main	lotus-storage-miner/run.go:66	Checking full node sync status
Worker 0: Target: [bafy2bzacea2qx4gqmilsx2vbxzdridygw4up4yxrordusm4bewewik2vxhx3q]      State: complete Height: 15777
Done!
2019-12-11T11:05:48.370+0800	INFO	badger	badger@v1.6.0-rc1/logger.go:46	All 1 tables opened in 0s
2019-12-11T11:05:48.612+0800	INFO	badger	badger@v1.6.0-rc1/logger.go:46	Replaying file id: 0 at offset: 98
2019-12-11T11:05:48.612+0800	INFO	badger	badger@v1.6.0-rc1/logger.go:46	Replay took: 12.758µs
2019-12-11T11:05:48.613+0800	INFO	p2pnode	lp2p/addrs.go:114	Swarm listening at: [/ip4/127.0.0.1/tcp/44665 /ip4/192.168.1.220/tcp/44665 /ip4/192.168.0.102/tcp/44665 /ip4/172.17.0.1/tcp/44665 /ip6/::1/tcp/35677]
2019-12-11T11:05:48.642+0800	INFO	build	build/paramfetch.go:121	Parameter file /var/tmp/filecoin-proof-parameters/v20-stacked-proof-of-replication-117839dacd1ef31e5968a6fd13bcd6fa86638d85c40c9241a1d07c2a954eb89b.vk is ok
 661.70 MiB / 5.29 GiB [==============>-------------------------------------------------------------------------------------------------------]  12.22% 5.22 MiB/s 15m07s2019-12-11T11:07:56.154+0800	INFO	basichost	basic/natmgr.go:96	DiscoverNAT error:no NAT found
 866.09 MiB / 5.29 GiB [==================>---------------------------------------------------------------------------------------------------]  15.99% 5.18 MiB/s 14m37s
```

你可以通过下面的方式查看矿工信息：
```sh
~/work/code/lotus$ ./lotus-storage-miner info
Miner: t02190
Sector Size: 256.0 MiB
Power: 0.0 B / 2.79 TiB (0.0000%)
Worker use:
	Local: 0 / 4 (+1 reserved)
	Remote: 0 / 0
Queues:
	AddPiece: 0
	PreCommit: 0
	Commit: 0
	Unseal: 0
Proving Period: Not Proving
Sectors:  map[Total:0]
```
## 存储挖矿
### 添加一个文件
```
#lotus client import ./hello.txt
bafkreifn3m3jpuezd67x6vkzf6wosxsootuslcaofmalxtnh72u6lxx644
```
### 查看本地的文件
```
#lotus client local
bafkreifn3m3jpuezd67x6vkzf6wosxsootuslcaofmalxtnh72u6lxx644 work/code/lotus/hello.txt 17 ok
```
### 查看所有可以存储数据的矿工
```
#./lotus state list-miners
# ./lotus state list-miners | wc -l //统计矿工数量
```
### 向某个矿工询问报价
```
# ./lotus client query-ask t02190
Ask: t02190
Price per GigaByte: 0.0000000005
```
### 存储文件
```
./lotus client deal bafkreihvzt6x34ersllpu23dskieuzvtmwnz3pqxd6deesyqysflm5zosu t02190 0.000005 300
bafyreihdzndfdwq3lokthihmjfbl4llwnvau7he3pqhdhnedzarofmzb4e： 成功返回交易id
WARN  main  lotus/main.go:72  routing: not found ： 表示矿工离线
WARN  main  lotus/main.go:72  failed to start deal: computing commP failed: generating CommP: Piece must be at least 127 bytes： 表示存储文件大小小于127字节
```

## 检索挖矿
### 根据DataId查找文件
```
./lotus client find bafkreihvzt6x34ersllpu23dskieuzvtmwnz3pqxd6deesyqysflm5zosu
LOCAL
RETRIEVAL t02190@12D3KooWEK9HuJtMQeujFnFtrHc41dSgW3vf1ifQmcXw4Kqvuq76-0.000000000000001292fil-646b
# RETRIEVAL <miner>@<miner peerId>-<deal funds>-<size>
### 检索出文件
```
./lotus client retrieve bafkreihvzt6x34ersllpu23dskieuzvtmwnz3pqxd6deesyqysflm5zosu ./test.txt
Success
```
现在这一步有问题，虽然返回状态是成功，但是文件是空的，官方正在解决这个问题

##在lotus节点搭建过程中，根据官方推荐顺便搭建了一个监控台，可以实时监控展示节点与链的运行状态。

### 环境依赖
- go (1.13 or higher)
- docker
- docker-compose

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

