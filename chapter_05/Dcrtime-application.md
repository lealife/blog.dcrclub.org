# Dcrtime使用及原理

dcrtime是dcr出的一个时间戳服务, 区块链有天然的时间戳属性, 每个block都会有一个timestamp字段来显示打包的时间, 而且这个时间是线性增长. 

通过交易的OP_RETURN字段, 你可以在交易中存储一些数据(长度有限, 80字节内), 因为长度有限, 主要也是文字为主, 比如 [这个服务](http://www.ibitlin.com) 就实现了在BTC/BCH上交易带数据美其名曰为比特币刻字. 因为一旦打包, 交易不可篡改, 不可还原, 所以交易上附带的数据也就一直存在区块链上. 通过这个功能, 我们用它作 产权保护, 存证等一些功能. 下面举两个例子证明它的作用.

## 时间戳服务举例

### 存证(拒绝404)
在网络上有些数据只会出现几天甚至几分钟后这个链接就成了404再也不能访问了. 这时就可以用区块链时间戳服务存储这些数据, 让这些数据永生. 再也不担心404. 这就是去中心化的好处, 没有强权.

### 产权保护
我辛辛苦苦写了一篇文章, 发出去后结果被改名, 我怎么证明这个文章是我第一个发出的? 你说你在网上有发出去的时间为证. 那其它人发出去的时间可以篡改啊, 没准它还反告你侵权呢?

当你写了文章后, 你可以这样做: 将该文章的hash数据 (hash算法将数据换成固定长度的输出, 不同的数据会有不同的hash. 为什么要存hash, 如果你存了原文不就谁都可以看到了, 别人就可以构造相同的hash, 记住原文是最重要的!!), 存到区块链上. 然后, 你公开的文章是你在原文中稍微修改的版本, 这就保证了别人看到的文章不能构造相同的hash.

此时, 你拥有原文+hash+交易(+交易上的时间信息), 这就表明, 你是在某个时间发布的文章.

别人不能构造相同的hash, 也就不能证明这个文章是它的.

别人拿你发的文章构造另一个hash, 再发到区块链上, 但时间是落后于你的, 所以你是第一发布者.

## 如何使用dcrtime
dcrtime现在主要为 [Politeia](https://proposals.decred.org/) 服务, 当然它是一个基础服务, 你也可以使用. 而且DCR官方公开的接口是免费的, 所以我们也可使用.

### 命令行使用
要使用命令行, 你得是一个程序员, 因为你自己还要编译它, 如果你不是, 请略过本小节.

dcrtime的代码在 https://github.com/decred/dcrtime 下载, 执行以下命令build (请确保你安装了go 1.11或最新版go)

```
cd cmd/dcrtime
go build

```
编译好后, 会有一个dcrtime二进制文件. 下面使用它.

将 dcrtime.go (你可以使用其它文件, 这里只是举个例子) hash后, 存到区块链上:

```
./dcrtime -v dcrtime.go 
617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 Upload dcrtime.go
617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 OK     dcrtime.go
Collection timestamp: 1551322800

```
注意, 这里的 Upload 不会上传这个文件的, 如果上传了哪还有什么安全可言, 它上传的是 这个文件的hash. 所以这里的 Upload 有歧义.

dcrtime.go hash的结果是 617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2, dcr的dcrtime官方服务已收到了要将 617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 打包的请求, 它会在每1个小时将所有的请求进行 Merkle 计算, 并把 Merkle root 上传到区块链上. (关于Merkle 请见本文的原理篇)

如果你再执行相同的命令:

```
$ ./dcrtime -v dcrtime.go 
617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 Upload dcrtime.go
617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 Exists dcrtime.go
Collection timestamp: 1551322800

```
**Exists** 表示这个请求已存在了.

大约过了一个小时(整点后), 我们执行` dcrtime -v hash` 来验证是否已上链:

```
./dcrtime -v 617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2
617718cfd5dda464c5b87ff5a6e0b4aceb6bd4f171c9bffa09fc06e8c18318c2 OK
  Chain Timestamp: 0
  Merkle Root    : 4b79e6184ddeca2a05073d2de1fb2dddcb1a83d889e9ee2ac0f3f87ad77aa17f
  TxID           : 003885d0e192c4a160244e776973caa0bec2ea9cdc88e50e74d02402524e7f67

```

表明, 请求已上链, tx Id 是 003885d0e192c4a160244e776973caa0bec2ea9cdc88e50e74d02402524e7f67, 可以在dcr区块链浏览器上看到这个[交易](https://explorer.dcrdata.org/tx/003885d0e192c4a160244e776973caa0bec2ea9cdc88e50e74d02402524e7f67)

![](https://teakki.com/file/image/5c775f60b1029f607605d3f0)

OP_RETURN 的是 Merkle root 数据.

### 网页使用
dcrtime的GUI工具, 官方已有一个项目 https://github.com/decred/authit 但还没上线, 如果你是程序员, 你可以下载运行. 我运行之后, 是这个样子:

![](https://teakki.com/file/image/5c7751bdb1029f607605cccb)

上传一个文件后, 提交:

![](https://teakki.com/file/image/5c7751f8b1029f607605cd08)

不是我想要的样子, 只能提交文件, 不能直接提交hash, 于是我也做了一个.
http://www.ibitlin.com/dcrtool#/dcrtime 大家可以直接体验. 支持上传文件, 纯文本, hash (Sha256是Merkle使用的hash算法)

![](https://teakki.com/file/image/5c775278b1029f607605cdb9)

![](https://teakki.com/file/image/5c7762c6b1029f607605d41c)

## 原理篇: 上链后如何验证
关于 Merkle tree 及 Merkle path验证的详细介绍, 请先看我整理的两个关于 [Merkle Tree](https://teakki.com/p/5ab07a111a37996365f10687) 文档.
dcrtime是每一个小时上链一次, 它将它这一小时收集的所有hash进行Merkle计算, 最终上链的是Merkle root.

上链的数据是 Merkle root, 而我只有文件的hash, 只有一个hash是无法构造出Merkle root, 既然构造不出, 也就无法验证我这个hash上链了.

所以, 我们必须还要用 Merkle path才行! 有了 Merkle path 我就可以构造出 Merkle root 也就证明了我的hash上链了.

所以, 在网页版中, 如果你的hash上链了, 会有一个**下载**按钮, 下载后里面会有 Merkle path, 大家一定要保管好这个Merkle path.

![](https://teakki.com/file/image/5c7754d2b1029f607605cfdb)

所以, 上链后, 你要保管的是:

* 原文件
* 下载的json文件, 主要包含信息
    * txId
    * Merkle root
    * Merkle path
    * NumLeaves hash数

## 原理篇: 再说下 Merkle path 验证
举个例子, Merkle path 来计算 Merkle root.

下载hash: e7738dd4557312683c44b5c98f985e87ed982e1a51c28ae29543b5adb7473101 的验证文件, 如下图输入这个hash, 并下载.

![](https://teakki.com/file/image/5c7763aab1029f607605d480)

这个文件里有

```
 "NumLeaves":6,
 "HashsHex":["83ba97af073afa648d5f973032ab0664060def37d631bf7760e96e77c2dd9850","e237d924213a4f3fa545966202440f5e3ebe0898000000000000000000000000","e7738dd4557312683c44b5c98f985e87ed982e1a51c28ae29543b5adb7473101"]

```

这几个hash就是供验证的 Merkle path, 注意, 这里面肯定有我们提交的 hash. 通过知道共**6个**hash 节点, 再加上这3个验证hash, 就可以构造出下面这棵 Merkle树. (注意, 必须要知道总共有几个hash节点, 不然构造不出Merkle树)

为了好描述, 按顺序, 我将这几个hash标号为 hash1, hash2, hash3

```
       Merkle root
        /       \
      1          5
    /   \       / \
   x     x      4  4 
  / \   / \    / \
 x   x  x  x   2  3(自己)

```

下面计算Merkle root

```
hash4 = hash(hash2+hash3) = sha256(e237d924213a4f3fa545966202440f5e3ebe0898000000000000000000000000e7738dd4557312683c44b5c98f985e87ed982e1a51c28ae29543b5adb7473101) = dda1e162618d9343c18234e02749023092c64370579c76396e53e9d57c55f8b1
hash5 = hash(hash4+hash4) = ff11d741ef102e4b0b6dfc92d46dba2467a6d072a0a3d7c5880ec20cc52f080c
Merkle root = hash(hash1+hash5) = db8b523064e73bacac3da2f37a02c920b7dc23b7c043b7cca932cf6a9daccdad

```
计算的结果就是上链的Merkle root, 验证成功.

关于计算hash的工具, 可以参考 http://www.ibitlin.com/dcrtool#/hash 

![](https://teakki.com/file/image/5c77819bb1029f607605e674)

## dcr官方dcrtime免费接口
仅两个接口, 也是 [http://www.ibitlin.com/dcrtool#/dcrtime](http://www.ibitlin.com/dcrtool#/dcrtime) 使用的接口:

* [https://time.decred.org:49152/v1/timestamp/](https://time.decred.org:49152/v1/timestamp/)
* [https://time.decred.org:49152/v1/verify/](https://time.decred.org:49152/v1/verify/)

参数参考:
[https://github.com/decred/dcrtime/blob/master/api/v1/api.md](https://github.com/decred/dcrtime/blob/master/api/v1/api.md)

## 后记

dcr官方的dcrtime接口是免费的, 因为上链要收手续费, 所以其实这是一个不免费的功能, 如果你也想搭一个dcrtime接口开放供他人使用, 你要考虑到这点.

1小时一次上链, 一次交易手续费 0.000262 DCR, 那么一年要消耗 24*365*0.000262 = 2.295 DCR

当然, 如果这个服务以后很流行了, 可以加一些额外功能, 比如: 

* 1小时一次太长了, 可以付费加快速度
* 不想和其它hash一起计算Merkle root, 我是土豪, 我只要我自己一个hash上链.

OP_RETURN 可以存任何数据, 这样一个很开放的功能, 在它上面可以作很多文章:

* 数据永存, 刻字服务
* 产权保护, 存证...
* 基于BTC的 omni 协议 也是用了它, USDT 大家熟悉吧, 就是在omni发币的
* 既然有 omni 协议, 你也基于OP_RETURN再创一个协议

Tips: 本文已上链, 大家不要乱复制我的来改, 见 [tx](https://explorer.dcrdata.org/tx/19077b836bae1b8306712314eb2256d245c8b7161e90198f3235cc877bed152a)

![](https://teakki.com/file/image/5c77835ab1029f607605e709)

原文链接: https://teakki.com/p/5c774a9ab1029f607605bc76
