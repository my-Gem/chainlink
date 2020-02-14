## 预言机(oracle)

```
在区块链领域，预言机被认为是可以为智能合约提供外部数据源的系统。从传统技术架构方面来看，预言机是连接智能合约与区块链外部世界的中间件（middleware），是区块链重要的基础设施，它的作用是为区块链上的智能合约提供数据信息的。
```



## chainlink

```
由于预言机是中心化的，需要从我们的数据中心获取数据，缺乏公信力，也不够安全等问题，故采用业内顶尖的智能合约去中心化预言机网络解决方案chainlink,
Chainlink可以赋予智能合约获取任意外部API数据的能力。通过创建了Chainlink项目，可以通过调用Chainlink开发库提供的方法来获取外部数据，将结果注入到您的智能合约中。

长久以来，区块链上运行的智能合约无法直接的与外部系统进行交流，这一瓶颈限制了智能合约应用场景。

如今，我们可以通过引入预言机（Oracle）的功能来解决这一问题，预言机可以为智能合约提供与外部世界的连接性。但是目前的预言机都是中心化的服务，这会给使用中心化预言机服务的智能合约带来单点故障的风险，使得智能合约的去中心化特性变得毫无意义。

为了解决这个问题，Chainlink开发了首个去中心化预言机项目，来向智能合约提供外部数据。有了这个武器，智能合约的安全性和确定性，与真实世界中发生的各类事件联系了起来。
```

PS:

chainlink的去中心化预言机oracle网络可以让您的同一类型的数据来源于多个数据源。以去中心化的方式从多个数据源消耗数据，您的合约能够进一步确保所提供数据的准确性。



### chainlink基本原理

```

```



### chainlink使用教程

```
https://chainlink-chinese.readme.io/docs
```



### 创建chainlink项目

1)首先需要引入Chainlinked.sol,然后继承Chainlinked合约

2)编写合约,示例代码如下:

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

  // 管理员调用创建Chainlink请求伴随the uint256 multiplier job
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

3)先需要在测试网络中测试合约(后期再去主网部署合约),测试合约即合约地址需要持有一定数量的LINK代币才能创建请求。需要消耗link代币，所以需要先去水龙头获取link代币(貌似需要翻墙)

4)创建请求时，您将需要为Chainlink节点指定需要在处理作业时用到的参数

```
// Creates a Chainlink request with the uint256 multiplier job and returns the requestId
function requestEthereumPrice(string _currency) public returns (bytes32 requestId) {
  // newRequest takes a JobID, a callback address, and callback function as input
  Chainlink.Request memory req = buildChainlinkRequest(UINT256_MUL_JOB, this, this.fulfillEthereumPrice.selector);
  // Adds a URL with the key "get" to the request parameters
  req.add("get", "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD,EUR,JPY");
  // Uses input param (dot-delimited string) as the "path" in the request parameters
  req.add("path", _currency);
  // Adds an integer with the key "times" to the request parameters
  req.addInt("times", 100);
  // Sends the request with 1 LINK to the oracle contract
  requestId = sendChainlinkRequest(req, 1 * LINK);
}
```





```
除了向单一的预言机发送请求，您还可以使用多个预言机来确认结果的准确性。使用多个预言机可以保证提供给合约的结果是准确的，从而高度保证您的智能合约正确触发。
每个预言机都支持相同类型的请求，如任务Task列所示，因此您可以将相同的参数发送到多个预言机，以确保您的合约不依赖于单个节点来执行。在 数据的请求与接收 查看相关例子。
本节我们列出了每个测试网络上运行的可用预言机及其预言机的Job ID。您需要一个Job ID才能从Chainlink请求数据。
```





```
我向区块链网络中发送一个Chainlink请求，您的请求者合约在链上向Oracle（预言机）合约发送一个事务,
Oracle合约跟踪来自请求者的LINK余额，在通过transferAndCall方法接收到LINK时，发出相关事件(event)，该事件会通知链下的Chainlink网络，一个新的请求已经启动，Chainlink节点会监听和处理区块链上的这些事件。
收到请求后，Chainlink就会去完成请求的相关工作，并将结果返回给Oracle合约，Oracle合约将会想节点运营方支付LINK，并将结果返回给消费者合约
```





```
开奖只能由管理员来触发执行,通过固定的开奖时间触发开奖合约,合约通过调用链下的开奖结果，写入区块链
```






个人总结

```
chainlink可以理解为一个中间件, 从中间件获取数据,中间件通过回调函数保存数据等等操作
```

