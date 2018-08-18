# Solidity-Vampire-ERC721
使用erc721 配合erc20 的宠物游戏

![01](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/erc721logo.png)

实现自己的ERC721宠物

你想拥有一个独一无二的宠物吗
以太坊 erc721 可以满足你，每一个个体都是独一无二的存在，举个例子：例如人民币（erc20） 一张坏了可以用另一张支付，这是可替换的。但是erc721可以类比成什么呢，世界上没有完全相同的树叶

如对erc721 和 erc20 概念不是很理解的同志请自行查阅资料，哈哈，不过多解释

本文章采用 erc721 + erc20 的方式 实现，erc20 代币 用作合约之中的流通货币，例如 宠物战斗，宠物繁殖，宠物竞拍的手续费，也可以直接采用erc20代币直接购买宠物！

本文引入了erc20 ，和 erc721 的接口 并对erc721 进行实现，而只调用部分erc20方法。

![erc721，erc20](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/erc721%EF%BC%8Cerc20.png)

接下来就带你一步一步实现自己的宠物
先大概构想一个宠物合约应有的功能，最基本的应该是繁殖，但不能无限繁殖，应先规定多久繁殖一次，本合约采用的是每个用户可获得一个宠物，随后获得宠物需要进食，然后有概率获得宠物，获得的宠物等级会在原等级相加，但是稀有度就看自己的运气了哈哈，一般来说等级越高，稀有度越稀有的宠物也就越珍贵。

创建宠物时会默认为宠物生成一个用户名，用户可以根据自己的喜好来更改这个用户名，但是需要支付很少一部分的手续费用

随后设想宠物的战斗功能，宠物有战斗力性质的存在，但是规定战斗胜利的基准是什么呢，是运气，没错就是运气，合约采用随机几率的方式来决定战斗输赢，赢了的一方将获得代币奖励，和战斗力增加，战斗也不可能无休止的进行，那怎么办呢，咱们接着为他加上冷却时间吧。输了也不可能一点惩罚没有，我们采用的是降低战斗力的方式。想让宠物战斗获取代币 你只需要支付一定量的代币（手续费），什么又是代币，没错有代币你就可以为所欲为！

然而这个可以让用户为所欲为的代币从哪里来呢？ 这点很重要

根据自己需要的场景来选择，如果你希望自己的代币 还有宠物 仅作为娱乐性质，那就可以 让用户每天签到获得代币，现在的合约里实现是前两周可以通过签到获得，后边需要再获得代币就需要联系 发币项目方了。可以通过这种方式让用户一直获得代币。然而如果你准备商用签到的方式可以保留，但是在此获取代币就需要用 人民币来购买喽，发币 上交易所 割韭菜 哈哈楼主还是不推荐这么干的。

要让宠物真正的流通起来 除了用户和用户的交易，最好的办法是什么，当然是拍卖，用户设定最低出价，拍卖时间由系统规定，然后你就可以坐等收币了，等竞拍完成，项目方会把你获得的代币，还有竞拍者获得的宠物双双交还。

接下来我们来具体实现它
首先定义一下 宠物要有的东西：

名字 ， DNA， 等级， 父亲是谁，战斗力，稀有度以及创建时间

此图包含了竞拍的结构体，留作接下来 实现竞拍所用，具体的东西有：参与人，参与人的金额， 初始金额， 是否完成竞拍宠物发放， 最后就是竞拍时间

![结构体](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E7%BB%93%E6%9E%84%E4%BD%93.png)

一个产品没有客户端是做不起来的，然后客户端怎么知道创建了一个宠物或者 你给宠物重新定义了一个名字呢

solidity中有个东西叫做监听 event。

试想一下咱们要返回给客户端的时候，创建了一个宠物，重命名了一个宠物，战斗获胜的宠物，还有竞拍的时候

![监听](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E7%9B%91%E5%90%AC.png)

看一下合约中具体要用到的变量，战斗的冷却时间，创建新僵尸的各种时间，还有各项费用。

![变量方法](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E5%8F%98%E9%87%8F%E6%96%B9%E6%B3%95.png)

因为区块链的不可更改特性，很多时候一个值都不能固定，例如签到获得的币，交易的手续费，这些都应该根据具体的现场场景来设置。所以我们实现了具体的方法

![设置变量](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E8%AE%BE%E7%BD%AE%E5%8F%98%E9%87%8F.png)

然后咱们要实现设置宠物的稀有度，dna等的私有方法，来看一下具体实现

![设置随机稀有度 dna等](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E8%AE%BE%E7%BD%AE%E9%9A%8F%E6%9C%BA%E7%A8%80%E6%9C%89%E5%BA%A6%20dna%E7%AD%89.png)

然后来看一下创建宠物的具体方法

![创建吸血鬼](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E5%88%9B%E5%BB%BA%E5%90%B8%E8%A1%80%E9%AC%BC.png)

战斗的具体实现：

![战斗](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E6%88%98%E6%96%97.png)

竞拍要分为 发起竞拍，竞拍中，竞拍结束的多步操作，咱们逐一实现

![竞拍](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/%E7%AB%9E%E6%8B%8D.png)

最后是针对erc721合约的具体实现，一个宠物本质就是erc721 (非同质货币)

![erc721合约实现部分](https://skyhuihui.github.io/skyhuihui.github.io/assets/img/2018-8-18-erc721%E5%AE%A0%E7%89%A9%E6%B8%B8%E6%88%8F/erc721%E5%90%88%E7%BA%A6%E5%AE%9E%E7%8E%B0%E9%83%A8%E5%88%86.png)

实现起来到这里就差不多结束了，可以根据自己的需求，进行功能的拓展，接下来会对合约进行逐步更新，修正bug等。

是不是已经心动了，下面是我的github实现地址， 欢迎fork，共同学习

https://github.com/skyhuihui/Solidity-Vampire-ERC721.git

如果您想支持我 可以像我的地址上转一些以太币 0x2207358972e37f663a5480dbaa09715e8b0fc4ff，什么你只有新潮的eos 没有以太币，放心放心，eos地址我也有 eosskyhuihui

算了开玩笑的了，哈哈， 您的转发就是对我最大的支持。
