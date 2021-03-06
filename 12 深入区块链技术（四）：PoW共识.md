# 12 深入区块链技术（四）：PoW共识

    上一篇文章中，我们谈到了区块链其实就是一种分布式系统。
    它在技术上并没有跳出分布式系统的理论框架，只是给出了一种不同于计算科学领域的解决方案。
    今天，我们就来重点聊聊区块链的解决方案：PoW共识机制。

## PoW工作量证明

    因为比特币采用了PoW共识机制，所以这个概念才得以被广泛传播。
    Pow全称Proof of Work，中文就是工足量证明。
    PoW共识机制其实是一种设计思路，而不是一种具体的实现。

    PoW机制其实早在1997年就被提出了，它早期多被应用在抵抗滥用软件服务的场景中。

    简单说明一下。

    为了防止垃圾消息泛滥，接收者并不直接接受来自任意发送者的消息，所以在一次有效的会话中，
    发送者需要计算一个按照规则约定难题的答案，发送者给接收者的同时，需要附带验证这个答案，
    如果这个答案有效，那么接收者才会接受这个消息。

    可以看出PoW的核心思路是提出一个难题，这个题目计算困难但是验证简单。
    这种特性我们称之为计算不对称性。

## 如何理解区块链PoW

    上面是一个一般的PoW的例子，接下来就说说区块链的PoW是如何设计的。

    在分析拜占庭将军问题的时候可以看出，如果所有节点在同一时刻发起提案，那么这个系统的记账过程
    会非常地复杂、混乱。为了降低具有提案权的节点数量，采用工作量证明不失为一个好方法。

    所以我们需要构造一个计算不对称的难题。
    这个难题在比特币中被选定为以SHA256算法计算一个目标哈希值，使得这个哈希值符合前N位全是0。

    举个例子，假定我们选定一个字符串“geekbang”，我们提出的题目就是这样的：

        选定一个数字，与给定的字符串连接起来使这个字符串的SHA256计算结果的前4位是0。

    这个数字被称作nonce，比如字符串“geekbang1234”，nonce就是1234。
    题目的解就是符合条件的nonce。

    我们以Python代码为例：

        #!/usr/bin/env python
        import hashlib

        def main():
            base_string = "geekbang"
            nonce = 10000
            count = 0
            while True:
                target_string = base_string + str(nonce)
                pow_hash = hashlib.sha256(target_string).hexdigest()
                count = count + 1
                if pow_hash.startswith("0000"):
                    print pow_hash
                    print "nonce: %s  scan times: %s" % (nonce, count)
                    break;
                nonce = nonce + 1
        
        if __name__ == '__main__':
            main()
        
    在代码中，我规定nonce从10000开始递增向上求解。

    计算结果如下：

        # 前4位是0
        0000250248f805c558bc28864a6bb6bf0c244d836a6b1a0c5078987aa219a404
        nonce: 68828  scan times: 58829
        # 前5位是0
        0000067fc247325064f685c32f8a079584b19106c5228b533f10c775638d454c
        nonce: 1241205  scan times: 1231206
        # 前7位是0
        00000003f41b126ec689b1a2da9e7d46d13d0fd1bece47983d53c5d32eb4ac90
        nonce: 165744821  scan times: 165734822

    可以看出，每次要求哈希结果的前N位多一个0，计算次数就多了很多倍。
    在例子中，要求前7位为0时，计算次数达到了1.6亿次。
    这个计算因为就是单纯的运算，所以也非常地消耗CPU。

    通过上述程序，希望你对区块链PoW机制有个直观的了解。
    由于结果只能暴力计算，而且可能的搜索空间非常巨大，作弊几乎不可能。
    另外，符合条件的nonce值也是均匀分布在整个空间中的，所以哈希是一个非常公平且粗暴的算法。

    以上代码的基本逻辑就是PoW挖矿过程，每次求得一个解就获得一次记账权，
    距离上一次打包至今未确认的交易，矿工就会一次性地将未确认的交易打包并广播，
    并从Coinbase获得奖励。

    实际挖矿的基本步骤如下：

        1.生成Coinbase交易，并与其他所有准备打包进区块的交易组成交易列表，
        并生成默克尔哈希
        2.把默克尔哈希及其他相关字段组装成区块头，将区块头（Block Header）作为工作量证明的输入，
        区块头中包含了前一区块的哈希，区块头一共80字节
        3.不停地变更区块头中的随机数nonce的值，并对每次变更后的区块头做双重SHA256运算，
        将结果与当前网络的目标值做对比，如果小于目标值，则解题成功

    如果更深程度去理解的话，PoW机制是将现实世界的物理资源转化成区块链虚拟资源的过程，
    这种转化为区块链提供了可信的前提。

## PoW挖矿的发展历程

    现在我们知道了，PoW的过程其实就是一个解难题的过程。

    在区块链的发展史上，PoW经历了大致两个阶段。
    分为早期的分散挖矿阶段和中心化矿池挖矿阶段。
    我们目前处于且将长期处于第二个阶段。

    早期分散挖矿是中本聪的愿景，期望是1CPU=1票，所以如果CPU挖矿，那么将会是非常理想化的情况。
    而现实的情况是，SHA256只需要非常简单的重复计算逻辑，而不需要复杂的控制逻辑。

    这样，显然交给GPU挖矿比CPU要合适的多。

    所以这个时期，就出现了GPU挖矿，它的效率是CPU的几十乃至上百倍。
    后来，还出现了专门用来挖矿的ASIC芯片。

    同期，我们也慢慢进入到了中心化挖矿阶段。中心化挖矿很好理解，算力越分散，竞争也就越激烈。
    如果某个节点挖到了矿，那么也就意味着其他节点做了无用功，甚至是负收益。

    那么怎么办呢？思路就是把分散的算力汇聚到一个池子里面，这个池子我们称作矿池。

    参与到矿池中后，相当于矿工把算力租给了矿池。联合的矿工越多，矿池的算力就越大。
    然后矿池获得奖励后，就按照比例分配给每一位参与的矿工。

    矿池作为一个中心节点，可以被矿工连接；然而在比特币全网看来，矿池节点本质上也只是一个全节点。
    它与其他全节点构成了比特币的点对点网络，只是它算力非常大而已。

## PoW挖矿算法分类与简介

    PoW挖矿算法大致分为两个大类，第一类叫做计算困难，第二类叫做内存困难。

    这两类的区别在于对于提供工作量证明的组件要求不同。我们知道计算机的组成部分分为计算单元和
    存储单元，而且一个计算机的瓶颈往往就在IO上。如果要优化大量的IO操作，
    可以通过撑大内存，制造大量数据处理过程的方法，来把工作量证明从计算单元转变为存储单元。

    那为什么要这样做呢？

    因为在PoW挖矿中心化以后，又出现了挖矿工具的改进。
    在GPU之后，人们尝试在FPGA进行SHA256的计算，结果效率成倍于GPU。

    FPGA出现不久，人们又开发出了ASIC专业芯片来计算SHA256，也就是我们常说的专业矿机。
    专业矿机的出现加剧了PoW挖矿中心化，专业矿机也几乎就成了数字货币的印钞机。

    新的数字货币开发者们为了防止情况重演，不断发明新的挖矿算法。
    有名的有Scrypt、X11、SHA-3等。不过这些依然是计算困难型的算法，都没有逃过出现专业矿机的命运。

    这里不得不提到以太坊的PoW挖矿算法：ETHASH。ETHASH是Dagger-Hashimoto的修改版本，
    它是典型的内存困难型挖矿算法。直到如今也没有出现专业的矿机。

    究其原因，是因为要求的计算资源是内存，也就是说矿工想要提高算力就要增加内存。
    而在专业矿机上加内存和直接给GPU加内存取得的收益是接近的，所以也就没有专业矿机的出现。

    这就在某种程度上缓解了算力中心化的问题。

## PoW的优势与劣势

    PoW共识的内在优势在于可以稳定币价。
    因为在PoW币种下，矿工的纯收益来自于Coinbase奖励减去设备和运营成本，
    成本会趋势矿工至少将币价维持在一个稳定水平，所以攻击者很难在短时间内获得大量算力来攻击主链。

    PoW共识的外在优势是目前它看起来依然是工业成熟度最高的区块共识算法，
    所以在用户信任度、矿工基础上都有很好的受众。

    PoW共识算法的最大缺点是非常消耗计算资源，耗电耗能源。
    每次产生新的区块，都无疑让一部分工足量证明白费了。

    目前看来这个是无解的，只要是PoW共识，都会有计算资源浪费的问题。

    从理论上看，PoW一直都有51%算力攻击问题，即攻击者只要购买全网51%以上的算力，
    即可发起“双花攻击”“重放攻击”等高收益攻击，这个问题目前没有解决方案。

    除了51%攻击，PoW共识还有自私挖矿的问题。自私挖矿是一种特殊的攻击类型，
    不会影响区块链正常运转，但是会形成矿霸，间接造成51%攻击。
    这个情况是曾经有发生过的。

## 总结

    今天我介绍了PoW工作量证明，并且使用Python演示了一个基于SHA256的挖矿算法过程。
    又介绍了发展历程和算法分类，最后提到了PoW的优势和缺陷。

    PoW共识机制是一种简单粗暴的共识算法，它不要求高质量的P2P网络资源，可以为公链提供
    稳定有效的记账者筛选机制。同时它也面临了挖矿中心化严重的问题，这也促使人们研究
    出了新的共识机制，我们留到下一篇讲解。
