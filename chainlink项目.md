

## 预言机(oracle)

```
在区块链领域，预言机被认为是可以为智能合约提供外部数据源的系统。从传统技术架构方面来看，预言机是连接智能合约与区块链外部世界的中间件（middleware），是区块链重要的基础设施，它的作用是为区块链上的智能合约提供数据信息的。
```



## 为什么使用 chainlink

```
长久以来，区块链上运行的智能合约无法直接的与外部系统进行交流，这一瓶颈限制了智能合约应用场景。

由于预言机是中心化的，需要从我们的数据中心获取数据，缺乏公信力，也不够安全等问题，故采用业内顶尖的智能合约去中心化预言机网络解决方案chainlink,Chainlink可以赋予智能合约获取任意外部API数据的能力。通过创建了Chainlink项目，可以通过调用Chainlink开发库提供的方法来获取外部数据，将结果注入到您的智能合约中。
如今，我们可以通过引入预言机（Oracle）的功能来解决这一问题，预言机可以为智能合约提供与外部世界的连接性。但是目前的预言机都是中心化的服务，这会给使用中心化预言机服务的智能合约带来单点故障的风险，使得智能合约的去中心化特性变得毫无意义。

为了解决这个问题，Chainlink开发了首个去中心化预言机项目，来向智能合约提供外部数据。有了这个武器，智能合约的安全性和确定性，与真实世界中发生的各类事件联系了起来。
```

PS:

chainlink的去中心化预言机oracle网络可以让您的同一类型的数据来源于多个数据源。以去中心化的方式从多个数据源消耗数据，您的合约能够进一步确保所提供数据的准确性。



## chainlink基本原理

```
Chainlink是一个去中心化的预言机项目，它的作用就是以最安全的方式向区块链提供现实世界中产生的数据。Chainlink在基本的预言机原理的实现方式之上，围绕LINK token通过经济激励建立了一个良性循环的生态系统。Chainlink预言机需要通过LINK token的转账来实现触发。
```



## chainlink浏览器

```
https://ropsten.explorer.chain.link/
```





## chainlink流程图

![avatar](http://pminer.oss-cn-shenzhen.aliyuncs.com/news/chainlink_liuchengtu.jpg)

## chainlink教程

```
https://docs.chain.link/docs
```

**PS：切记不要使用中文版本的chainlink文档，因为跑的一些示例代码提供的东西有误,直接使用英文版本**

PS:适配器就是自定义数据解析方式 

```
add：添加一个 string
addBytes：添加 bytes
addInt：添加一个整型int256
addUint：添加一个非负整型uint256
addStringArray：添加字符串数组
```

我们指定`"get"`为重点[HTTPGET](https://docs.chain.link/docs/adapters#section-httpget)适配器，`"path"`为重点[JsonParse](https://docs.chain.link/docs/adapters#section-jsonparse)适配器，以及`"times"`为重点[乘](https://docs.chain.link/docs/adapters#section-multiply)适配器。

PS:[适配器地址](https://docs.chain.link/docs/adapters)



##  创建chainlink项目

### 步骤一:

首先需要引入Chainlinked.sol,然后继承 ChainlinkClient合约

### 步骤二:

编写合约,示例代码如下:

```
pragma solidity 0.4.24;

import "chainlink/contracts/ChainlinkClient.sol";

contract ChainlinkExample is ChainlinkClient {
  
  //LINK是开发助手库中定义的uint256类型的常量，用来表示LINK代币可分的最小额,与10^18等价。
  //定义1个请求需要消耗1个link代币
  uint256 constant private ORACLE_PAYMENT = 1 * LINK;
  // 定义不同网络下 Chainlink uint256 multiplier JobID，JobID应该根据网络不同而不同
  bytes32 constant UINT256_MUL_JOB = bytes32("493610cff14346f786f88ed791ab7704");

  //定义一个变量用于存储来自Chainlink oracle的数据
  uint256 public currentPrice;
  //定义一个管理员
  address public owner;

  //构造函数
  constructor() public {
    //设置在ropsten测试网中LINK代币合约的地址，用于向oracle合约发起请求。在不同的测试网络上LINK代币合约地址都是不一样的。
    setChainlinkToken(0x20fE562d797A42Dcb3399062AE9546cd06f63280);
    // Set the address of the oracle to create requests to
    //设置在ropsten测试网中oracle（预言机）合约的存储地址，也就是我们需要通信的oracle地址
    setChainlinkOracle(0xc99B3D447826532722E41bc36e644ba3479E4365);
    //赋予管理员权限
    owner = msg.sender;
  }

  // 管理员调用创建Chainlink请求
  function requestEthereumPrice()  public  onlyOwner{
    // 实例化一个Chainlink请求。返回值Request是一个结构体，包含了发送给oracle合约的所需参数。buildChainlinkRequest需要三个参数，一个是JobID,一个是回调地址，用于接收结果；最后一个是回调函数签名，用于调用回调函数。
    Chainlink.Request memory req = buildChainlinkRequest(UINT256_MUL_JOB, this, this.fulfill.selector);
    // 定义get键对应的url地址
    req.add("get", "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD");
    // Uses input param (dot-delimited string) as the "path" in the request parameters
    req.add("path", "USD");
    // Adds an integer with the key "times" to the request parameters
    req.addInt("times", 100);
    //向存储地址发送请求数据存储在变量chainlinkOracleAddress中
    //向存储的Oracle合约地址发送请求数据。它需要一个Chainlink.Request类型以及需要发送的LINK金额作为参数。请求是经过序列化过的，通过LINK代币合约的transferAndCall方法，向chainlinkOracleAddress中存储的地址调用oracleRequest方法。
    sendChainlinkRequest(req, ORACLE_PAYMENT);
  }

  // 通过回调函数用于接收数据
  function fulfill(bytes32 _requestId, uint256 _price) public
    // recordChainlinkFulfillment是在履约函数中使用，确保调用方和requestId有效。这避免了Chainlink客户端回调函数被恶意调用。
    recordChainlinkFulfillment(_requestId)
  {
    //将链下获取的数据赋给变量
    currentPrice = _price;
  }
  
  // withdrawLink allows the owner to withdraw any extra LINK on the contract
  //允管理员撤销合约上任何额外需要消耗link的操作
  function withdrawLink() public onlyOwner{
    //chainlinkTokenAddress方法返回存储的Chainlink代币地址。代币地址这个变量是受保护的成员，所以只能通过getter和setter来访问。
    LinkTokenInterface link = LinkTokenInterface(chainlinkTokenAddress());
    //判断管理员的link转到当前合约是否成功
    require(link.transfer(msg.sender, link.balanceOf(address(this))), "Unable to transfer");
  }
  
  //定义管理员
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }
}
```



### 步骤三:

先需要在测试网络中测试合约(后期再去主网部署合约),测试合约即合约地址需要持有一定数量的LINK代币才能创建请求。需要消耗link代币，所以需要先去水龙头获取[link代币](https://ropsten.chain.link/)(貌似需要翻墙)，[LINK代币在ropsten测试网地址](0x20fE562d797A42Dcb3399062AE9546cd06f63280)，我们通过metamask转到消费者合约去



### 步骤四:

**先向已部署的消费者合约转10Link,一定要消费者合约必须要有LINK代币**，然后再调用requestEthereumPrice() 即可,**记得调用的job_id不是一样的[,参考](https://docs.chain.link/docs/use-your-first-contract)**,然后再点击currentPrice就可以看到链下获取的currentPrice价格了，没调用一次requestEthereumPrice()，消费者合约就会消耗1LINK

**PS：对应的测试网oracle 地址和oracle job_id，不然无法获取数据**，[示例](https://docs.chain.link/docs/use-your-first-contract)







## 个人总结

```
正式网络中,每发送一个请求到oracle合约,可能会消耗0.1LINK,确保测试合约必须拥有LINK代币,不然无法获取到数据，当然chainlink节点也可以从多个地方获取数据
在实际测试中,发送的请求到以太坊ropsten网络中的oracle合约中可以理解为合约交互需要3个区块的确认
```

