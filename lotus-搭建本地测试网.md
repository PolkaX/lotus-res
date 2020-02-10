## 搭建一个本地的lotus测试网
本篇讲解如何启动一个本地测试网络并创建新矿工，刷脏数据提升新矿工算力，扩充新矿工的存储。
### 搭建多个矿工节点的测试网
#### 从单节点测试网添加节点
filecoin 官方提供了一个单节点[搭建教程](https://docs.lotu.sh/en+setup-local-dev-net)，以下操作过程是在官方单节点搭建教程基础上做的扩展
#### 1. 编译debug版本的二进制

```
make debug
```

#### 2. 初始化第一个存储节点的扇区

```
$ ./lotus-seed --sectorbuilder-dir=/home/fy/work/node/local-lotus/storage1 pre-seal --sector-size 1024 --num-sectors 2
```
#### 3. 启动两个lotus节点

```
./lotus --repo=/home/fy/work/node/local-lotus/lotus1 daemon --lotus-make-random-genesis=dev.gen --genesis-presealed-sectors=/home/fy/work/node/local-lotus/storage1/pre-seal-t0101.json --bootstrap=false --api 30000

./lotus --repo=/home/fy/work/node/local-lotus/lotus2 daemon --api 30001 --genesis-presealed-sectors=/home/fy/work/node/local-lotus/storage1/pre-seal-t0101.json --bootstrap=false
```
此时两个lotus节点并没有互相链接,我们需要使用命令查询其中一个节点的bootnodes，然后用另一个节点连接他。

#### 4. 查看其中一个节点的监听地址,使用命令是另一个节点连接上它。

```
$ ./lotus --repo=/home/fy/work/node/local-lotus/lotus1 net listen
/ip4/127.0.0.1/tcp/38969/p2p/12D3KooWJbv5t4h7PSEk7TYKWYqYB1fzcFKfuwLkmNwwLdd96jep
...
/ip4/172.17.0.1/tcp/38969/p2p/12D3KooWJbv5t4h7PSEk7TYKWYqYB1fzcFKfuwLkmNwwLdd96jep
/ip6/::1/tcp/35503/p2p/12D3KooWJbv5t4h7PSEk7TYKWYqYB1fzcFKfuwLkmNwwLdd96jep

```
将bootnodes发送给另外一个节点

```
$ ./lotus --repo=/home/fy/work/node/local-lotus/miner2 net connect /ip4/127.0.0.1/tcp/38969/p2p/12D3KooWJbv5t4h7PSEk7TYKWYqYB1fzcFKfuwLkmNwwLdd96jep
connect 12D3KooWJbv5t4h7PSEk7TYKWYqYB1fzcFKfuwLkmNwwLdd96jep: success
```

[注]. 在第6步和第7步会看到日志打印：
```
2020-02-1 T17:52:47.177+0800	WARN	hello	hello/hello.go:78	other peer has different genesis! (bafy2bzaceaxm23epjsmh75yvzcecsrbavlmkcxnva66bkdebdcnyw3bjrc74u)
```
如果能个确保前面的几步都操作无误，这条日志是说与网络中其他链不是同一个genesis

#### 5. 初始化并启动第一个创世矿工，t0101是代码中默认的矿工，网络启动就有算力
初始化矿工：
```
env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus-storage-miner init --genesis-miner --actor=t0101 --pre-sealed-sectors=/home/fy/work/node/local-lotus/storage1 --nosync=true --sector-size=1024

2020-01-19T11:34:43.723+0800	INFO	main	lotus-storage-miner/init.go:89	Initializing lotus storage miner
2020-01-19T11:34:43.723+0800	INFO	main	lotus-storage-miner/init.go:98	Checking proof parameters
2020-01-19T11:34:43.723+0800	INFO	build	
...
2020-01-19T11:34:43.724+0800	INFO	build	go-paramfetch@v0.0.1/paramfetch.go:162	GET https://ipfs.io/ipfs/QmXjSSnMUnc7EjQBYtTHhvLU3kXJTbUyhVhJRSTRehh186
2020-01-19T11:34:43.893+0800	INFO	build	go-paramfetch@v0.0.1/paramfetch.go:127	Parameter file /var/tmp/filecoin-proof-parameters/v20-proof-of-spacetime-election-a4e18190d4b4657ba1b4d08a341871b2a6f398e327cb9951b28ab141fbdbf49d.params is ok
 338.63 MiB / 2.48 GiB [========>--------------------------------------------------------]  13.32% 3.70 MiB/s 09m53s
 
```
如日志所示，首次启动需要从从filecoin下载参数证明文件，这一步会需要比较长的时间。

启动矿工：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner run --nosync --api 20001
```

t0101矿工出块之后，t0101矿工账户里边就已经有余额了
查看余额：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus wallet balance
98672.255871674487797136
```
查看初始化矿工的算力：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus-storage-miner info
Miner: t0101
Sector Size: 1 KiB
Power: 2 KiB / 2 KiB (100.0000%)
	Committed: 2 KiB
	Proving: 2 KiB
Worker use:
	Local: 0 / 4 (+1 reserved)
	Remote: 0 / 0
Queues:
	AddPiece: 0
	PreCommit: 0
	Commit: 0
	Unseal: 0
PoSt Submissions:
	Previous: Epoch 439 (1 block(s), ~0m 30s ago)
	Fallback: Epoch 449 (in 11 blocks, ~5m 30s)
	Deadline: Epoch 459 (in 21 blocks, ~10m 30s)
Sectors:  map[Proving:2 Total:2]
```
创建一个bls类型地址：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner wallet new bls
t3qmkputiskghjga4jw7vyuwxvwjq4xzdspqs2q7muddzn27bakstanpbgyeipwfarvj4btbe2e6mqhc5jm5fq
```
给刚创建的地址转账：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus send t3qmkputiskghjga4jw7vyuwxvwjq4xzdspqs2q7muddzn27bakstanpbgyeipwfarvj4btbe2e6mqhc5jm5fq 10000
```
#### 6. 创建第二个矿工

创建并初始化一个新矿工：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner init --create-worker-key=true  --sector-size=1024 --pre-sealed-sectors=/home/fy/work/node/local-lotus/storage2 --nosync
```
查看矿工列表：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner state list-miners
```

启动第二个矿工：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner run --nosync --api 20001
```
[注]：默认端口被占用，需要指定端口。

查看算力：
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner info
Miner: t01002 # 矿工 ID
Sector Size: 1 KiB #扇区大小
Power: 0 B / 2 KiB (0.0000%) # 当前矿工的存储算力
	Committed: 0 B
	Proving: 0 B
Worker use:
	Local: 0 / 4 (+1 reserved) #本地矿工数量
	Remote: 0 / 0   #远端矿工数量
Queues:
	AddPiece: 0
	PreCommit: 0
	Commit: 0
	Unseal: 0
Proving Period: Not Proving
Sectors:  map[Total:0]
```
此时新增的第二个矿工没有算力， 我们需要向t01002矿工存储一些数据，t01002矿工才可以拥有算力。

#### 7. 存储文件
添加一个文件（文件必须大于256字节,小于初始化时设置的扇区大小）:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus client import ./hello.txt
bafkreifragqmiyh5qxh23iqra552xf54j5f4zlocjw5uqqhfo4llcdckqa
```
向某个矿工询价，在这里我们要往t01002矿工存储数据:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus client query-ask t01002
Ask: t01002
Price per GiB: 0.0000000005

```
发起一笔存储交易:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus1 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner1 ./lotus client deal bafkreifragqmiyh5qxh23iqra552xf54j5f4zlocjw5uqqhfo4llcdckqa t01002 0.000005 300
bafyreibddlukjniptmsxmre7ygnqqu6em6uisdreywyilg2ev2diej34aq
```
[注意]：虽然查询的每字节价格是0.0000000005，但是实际付款时价格需要高出一点，否则交易会执行失败。

存储交易被矿工执行以后，会经历一下几个状态：
打包:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus-storage-miner info
Miner: t01002
Sector Size: 1 KiB
Power: 0 B / 2 KiB (0.0000%)
	Committed: 0 B
	Proving: 0 B
Worker use
	Local: 0 / 4 (+1 reserved)
	Remote: 0 / 0
Queues:
	AddPiece: 0
	PreCommit: 0
	Commit: 0
	Unseal: 0
Proving Period: Not Proving
Sectors:  map[Packing:1 Total:1]


...
...
Sectors:  map[Packing:0 Total:1 WaitSeed:1]
...
...
Sectors:  map[Committing:1 Packing:0 Total:1]
...
...
Sectors:  map[Packing:0 Proving:1 Total:1]
```
封包速度跟机器配置相关。

* 矿工接收到客户端发送过来的数据包之后调用lotus*-seal-worker的api立即进行封包操作，完成封包之后进行 committing sector（提交打包好的扇区) 
    
* 运行时空证明(running PoSts)
    
* 请求将 PoSts 上链(Waiting for post bafxxx to appear on chain) 
    
* 上链成功，返回上链区块高度("height": 618)
    
* 当扇区密封并成功上链之后矿工获得算力就可以参与出块*

#### 9. 检索数据
通过cid查找数据在哪里存，及大小等信息:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus client find bafyreibddlukjniptmsxmre7ygnqqu6em6uisdreywyilg2ev2diej34aq
```
找回数据:
```
$ env LOTUS_PATH=/home/fy/work/node/local-lotus/lotus2 LOTUS_STORAGE_PATH=/home/fy/work/node/local-lotus/miner2 ./lotus client retrieve bafkreifgxbfutlcrbfnwpk5gx6o5of4mpleqvbkt5thphvbnsg6mnasp3q back.tx
```
### 以上过程演示了如何一步步搭建测试网络，接下来是官方测试网启动脚本
#### 使用官方提供的脚本
#### 1. 官方也提供了脚本 lotus/scripts/init-network.shs
