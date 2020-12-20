# <center>Tron</center>

- [下载与资源：](#下载与资源)
- [常用工具](#常用工具)
- [Tronbox](#tronbox)
- [tronbox/config.js](#tronboxconfigjs)
- [与以太坊有区别的特殊常量](#与以太坊有区别的特殊常量)
- [tronWeb](#tronweb)
- [TRX转帐](#trx转帐)
- [与合约交互](#与合约交互)
- [单位转换](#单位转换)
- [地址格式说明](#地址格式说明)
- [查询事件](#查询事件)
- [wei单位的转换](#wei单位的转换)
- [录入通证](#录入通证)
- [feeLimit](#feelimit)
- [调用合约的三种方法](#调用合约的三种方法)
- [justswap factory](#justswap-factory)
- [监听事件](#监听事件)


## 下载与资源：

[开发文档 ｜](https://cn.developers.tron.network/docs)
[中文文档 ｜](https://tronprotocol.github.io/documentation-zh/)
[开发入门 ｜](https://m.qukuaiwang.com.cn/news/14218.html)
[参考](https://blog.csdn.net/venus1100/article/details/84757739)

## 常用工具
Tron-Web：JavaScript接口，用于提供常用的账户，地址，转账，合约相关操作。相当于Ethereum的web3js。
tron-box：提供合约编译，部署，测试的命令行工具。作用相当于Ethereum的truffle工具链。
tronLink，tronPay: 提供浏览器环境的钱包插件，为dapp提供便利和安全的执行环境，相当于Ethereum的MetaMask，Scatter。
tron-grid：社区维护的主网和测试网HTTP API接口，相当于Ethereum社区中的Infura。
Tron-IDE: 一款帮助开发者开发智能合约的在线编辑器。它具有模块化的特点，以插件的方式为开发者提供智能合约的编辑，编译，部署，调试等功能。

## Tronbox
Tronbox v2.7.13
```
cnpm install -g tronbox

tronbox -v

Usage: tronbox <command> [options]
Commands:
  init     Initialize new and empty tronBox project
  compile  Compile contract source files
  migrate  Run migrations to deploy contracts
  deploy   (alias for migrate)
  build    Execute build pipeline (if configuration present)
  test     Run JavaScript and Solidity tests
  console  Run a console with contract abstractions and commands available
  watch    Watch filesystem for changes and rebuild the project automatically
  serve    Serve the build directory on localhost and watch for changes
  unbox    Download a tronbox Box, a pre-built tronbox project
  version  Show version number and exit

tronbox compile 编译

//Shasta测试网
tronbox migrate --network Shasta  //布署上线
tronbox migrate --network Shasta  --reset
https://shasta.tronscan.org/#/  //区块浏览器

//主网
tronbox migrate --network Mainnet --reset
https://tronscan.org/#/   //区块浏览器

Running migration: 1_initial_migration.js
  Replacing TronSteem...
  TronSteem:
    (base58) TRMAPytzETkYRsr4ug5Fts66XvxyTed3SK
    (hex) 41a8b0dd11fab3737ba5be4f7f72a2a4363f98d5b4
Saving artifacts...

— API:
https://api.shasta.trongrid.io
https://api.trongrid.io/
```

## tronbox/config.js
```js
module.exports = {
  networks: {
    Shasta: {
      from: 'TDEQDahvDBQiByPMWUK3gP4iKPQqywnVVk',
      privateKey: '188xxxxxx',
      consume_user_resource_percent: 30,
      fee_limit: 300000000,
      fullHost: "https://api.shasta.trongrid.io",
      // fullNode: "https://api.shasta.trongrid.io",
      solidityNode: "https://api.shasta.trongrid.io",
      eventServer:  "https://api.shasta.trongrid.io",
      network_id: "*" // Match any network id
    },

    Mainnet: {
      from: 'TDEQDahvDBQiByPMWUK3gP4iKPQqywnVVk',
      privateKey: 'eb3xxxxxxxx',
      consume_user_resource_percent: 30,
      fee_limit: 300000000,
      fullHost: "https://api.trongrid.io",
      solidityNode: "https://api.trongrid.io",
      eventServer:  "https://api.trongrid.io",
      network_id: "*" // Match any network id
    },

    Mainnet2: { //另一种写法
      from: 'TDEQDahvDBQiByPMWUK3gP4iKPQqywnVVk',
      privateKey: 'eb3xxxxxxx',
      userFeePercentage: 100,
      feeLimit: 3e8,
      fullHost: 'https://api.trongrid.io',
      network_id: '1'
    },
},

compilers: {
    solc: {
      version: '0.5.8'
    }
  },
};
```

## 与以太坊有区别的特殊常量
货币
类似于solidity对ether的支持，波场虚拟机的代码支持的货币单位有trx和sun，其中1trx = 1000000 sun，大小写敏感，只支持小写。目前Tron-IDE支持trx和sun，在remix中，不支持trx和sun，如果使用ether、finney等单位时，注意换算(可能会发生溢出错误)。 我们推荐使用Tron-IDE代替remix进行tron智能合约的编写。

区块相关
block.blockhash(uint blockNumber) returns (bytes32)：指定区块的区块哈希——仅可用于最新的 256 个区块且不包括当前区块；而 blocks 从 0.4.22 版本开始已经不推荐使用，由 blockhash(uint blockNumber) 代替
block.coinbase (address): 产当前区块的超级节点地址
block.difficulty (uint): 当前区块难度，波场不推荐使用，设置恒为0
block.gaslimit (uint): 当前区块 gas 限额，波场暂时不支持使用, 暂时设置为0
block.number (uint): 当前区块号
block.timestamp (uint): 当前区块以秒计的时间戳
gasleft() returns (uint256)：剩余的 gas
msg.data (bytes): 完整的 calldata
msg.gas (uint): 剩余 gas - 自 0.4.21 版本开始已经不推荐使用，由 gesleft() 代替
msg.sender (address): 消息发送者（当前调用）
msg.sig (bytes4): calldata 的前 4 字节（也就是函数标识符）
msg.value (uint): 随消息发送的 sun 的数量
now (uint): 目前区块时间戳（block.timestamp）
tx.gasprice (uint): 交易的 gas 价格，波场不推荐使用，设置值恒为0
tx.origin (address): 交易发起者

## tronWeb
cnpm install tronweb --save

注意: window.tronWeb 是页面中最后加载的！

```js
mounted() {
  let that = this
  async function main(){
    await that.sleep()  //等待6秒
    //tronlink
    if (window.tronWeb) {
      // console.log(22, "tronlink is ok! login")
      that.addr = window.tronWeb.defaultAddress.base58
    }else{
      that.tronlinkFlag = false
    }
    that.isLoading = false
    that.loadingFlag = true

    //如果没有获取到则再获取一次
    if(!that.tronlinkFlag){
      that.isLoading = true
      await that.sleep()
      //tronlink
      if (window.tronWeb) {
        console.log(522, "tronlink is ok! login")
        that.addr = window.tronWeb.defaultAddress.base58
      }else{
        let link2 = 'TronLink: https://www.tronlink.org'
        that.showMask = true
        that.maskInfo = "出错啦！请安装TronLink！\n"+link2
        that.tronErrorFlag = true
        that.isLoading = false
      }
    }
    that.isLoading = false
    that.loadingFlag = true
  }
  main()
}
```

## TRX转帐
```js
async transfer(){
  console.log(161222, "Yes, catch it:", window.tronWeb)
  let tronweb = window.tronWeb
  let sendTransaction = await tronweb.trx.sendTransaction("TVJ1Waucj32mSMuFRab7kn73Pm6KyZGBmg", 1000)
  console.log(566, sendTransaction)
}
```

## 与合约交互
```js
let res = await that.axios.get('/Test.json')
// console.log(66744, res)
let data = res.data
let tronWeb = window.tronWeb
let address = tronWeb.address.fromHex(data.networks['*'].address)  //转变成base58
let contract = tronWeb.contract(data.abi, address)
let s = await contract.a().call() //查询方法view
let stringx = 'WSTEEM是ERC20代币，旨在打通STEEM和以太坊网络的通道，使资产自由流通！'
let s2 = await contract.set(stringx).send() //更改方法
```

## 单位转换
波场中得到的uint数据都是16进制（hex,如：[{"_hex":"0x0f4240"}]）,如果在js中做加减乘除可以自动转换成10进制，也可以自动调整。
1 trx = 1000000 sun
```js
uint public b = 1 trx;

let  b = await contract.b().call() //查询方法view
console.log(333, 'b', tronWeb.toBigNumber(b).toFixed())
//333 "b" "1000000"

tronWeb.fromSun()
> tronWeb.fromSun("1000000")
'1'
本地创建的得到 object
浏览器创建的得到 string

tronWeb.toSun()
tronWeb.toSun(10)
>"10000000"
```

## 地址格式说明
用公钥P作为输入，计算SHA3得到结果H, 这里公钥长度为64字节，SHA3选用Keccak256。
取H的最后20字节，在前面填充一个字节0x41得到address。

对address进行basecheck计算得到最终地址，所有地址的第一个字符为T。

其中basecheck的计算过程为：首先对address计算sha256得到h1，再对h1计算sha256得到h2，取其前4字节作为check填充到address之后得到address||check，对其进行base58编码得到最终结果。

我们用的字符映射表为：
ALPHABET = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"

在波场中地址格式为："TPY1jv2wrnEHpQAxfQ7BbpyKsDQRZTYDPX"
16进制表示："4194cdf62db7f02319ec3b9a4009ba60854a896072"
转换函数：tronWeb.address.fromHex(address) //16进制转base58
         this.tronWeb.address.toHex(dd)  //base58转16进制
eg:
   NutboxsPeanuts:
    (base58) TYuDs3D3RZVXKhfPocYSyDBV8rES3MEiSs
    (hex) 41fb89f2d17db9b45988645908a623855d956d5ba9    

注意：波场区块链底层仍是使用的16进制的方式，但是与前端交互的却是使用的base58


## 查询事件
```js
//getEventResult
let s = await this.tronWeb.getEventResult(contract, {eventName:'Transfer', size:2})
tronWeb.getEventResult(contractAddress, {}, callback);
{}中的参数：
  sinceTimestamp：	Filter for events since certain timestamp.
  eventName：	Name of the event to filter by.
  blockNumber：	Specific block number to query
  size：	maximum number returned
  onlyConfirmed：	If set to true, only returns confirmed transactions.
  onlyUnconfirmed：	If set to true, only returns unconfirmed transactions.
  fingerprint：	The fingerprint field appears in the last data of the previous query. After specifying the corresponding field content this time, subsequent data will be returned. If there is no this field in the last data of the query, it means that there is no more data

eg:
let contract = "TU7Rv3sGrf1inemaSB5wENMFBgJTJwmiU5"
let s = await this.tronWeb2.getEventResult(contract, {eventName:'SteemToTsteem', size:4})
res:
  block: 8717043
  contract: "TU7Rv3sGrf1inemaSB5wENMFBgJTJwmiU5"
  name: "SteemToTsteem"
  resourceNode: "solidityNode"
  result:
  0: "timool"
  1: "0xd3f8b3766b8e50ae5b1980e3967c999c35c0af29"
  2: "100000000000000000"
  amount: "100000000000000000"
  steemAccount: "timool"
  user: "0xd3f8b3766b8e50ae5b1980e3967c999c35c0af29"
  __proto__: Object
  timestamp: 1602285000000
  transaction: "786f07e0bc8521beef2933ed71772dfae6b8dde44a2cc0ea9617b0639227393d"


//getEventByTransactionID
let id = "786f07e0bc8521beef2933ed71772dfae6b8dde44a2cc0ea9617b0639227393d"
let s = await this.tronWeb2.getEventByTransactionID(id)
res:
  block: 8717043
  contract: "TU7Rv3sGrf1inemaSB5wENMFBgJTJwmiU5"
  name: "Transfer"
  resourceNode: "solidityNode"
  result:
  0: "0x0000000000000000000000000000000000000000"
  1: "0xd3f8b3766b8e50ae5b1980e3967c999c35c0af29"
  2: "100000000000000000"
  from: "0x0000000000000000000000000000000000000000"
  to: "0xd3f8b3766b8e50ae5b1980e3967c999c35c0af29"
  value: "100000000000000000"
  __proto__: Object
  timestamp: 1602285000000
  transaction: "786f07e0bc8521beef2933ed71772dfae6b8dde44a2cc0ea9617b0639227393d"
```


## wei单位的转换
```js
const dataToWei = function(data){
  let a = data * 1e18
  return this.tronWeb2.toBigNumber(a).toFixed()
}

export {dataToWei}

const dataFromWei = function(data){
  let b = data * 1e-18
  return this.tronWeb2.toBigNumber(b).toFixed(6)
}

export {dataFromWei}
```

## 录入通证
[tronscan](https://tronscan.org/#/tokens/create/Record)

![tronscan.jpg](https://steemjiang.com:8081/ipfs/QmUpddYMHAAC9JXjwPDNjQWEPxdschUHjHQQnUjLzi3d2J)

TRC20通证和ERC20通证类似，不过因为编译的版本不一致，导致在波场中OpenZeppelin的库无法使用，只能使用原文件。通读下通证的代码，函数基本一致，倒也不用太过担心。这里是[PNUT合约](https://github.com/nutbox-dao/nutbox-contract/blob/master/contracts/NutboxPeanuts.sol)，大家可以参考。

TRC20通证合约布署上线后，还需要在tronscan中录入，然后才能上线justswap和poloni这样的DEX。如上图所示，几个输入的参数要和通证合约保持一致，否则无法录入成功！

几点说明：
1. 像PNUT这样是通过挖矿来分发的，并没有初始量。但tronscan中录入必须有这项参数，所以，可以设初始总量为1。tronscan网络会每隔一段时间（大约是24小时）对这些参数进行更新。当你第二天去查PNUT的总量时，这个参数会更新。
2. 精度（小数位）最好设为6，而不是18！波场这边的最小单位是sun, 1trx = 1000000sun。并且poloni也只接受6位精度的通证。
3. 在媒体和联系尽量填全，有助于提升大家对于通证的了解和信心。


## feeLimit
最多只花费多少个trx，单位是sun

[文档说明](https://developers.tron.network/reference#methodwatch)

```js
let instance = this.$store.state.nutPoolInstance
await instance.withdrawPeanuts().send({feeLimit:20_000_000})

async test10(){
  let instance = this.$store.state.steemInstance
  let a = await instance.test10().send({
    feeLimit:20_000_000,
    callValue:0,
    shouldPollResponse:true
  })
},
```

## 调用合约的三种方法
```js
//方法一
let abi = [...];       
let instance = await tronWeb.contract(abi,'contractAddress')
let result = await instance.function_name(para1,para2,...).call()
let result2 = await instance.function_name(para1,para2,...).send()

//方法二
可以省略abi这个参数
let instance = await tronWeb.contract().at('contractAddress');
let result = await instance.function_name(para1,para2,...).call()
let result2 = await instance.function_name(para1,para2,...).send()

//方法三
tronWeb.transactionBuilder.triggerConstantContract(contractAddress,functions, options,parameter,issuerAddress)
eg1:
let parameter = []
let transaction = await tronWeb.transactionBuilder.triggerSmartContract("419e62be7f4f103c36507cb2a753418791b1cdc182", "name()", {},
    parameter,"417946F66D0FC67924DA0AC9936183AB3B07C81126")
eg2:
let parameter1 =  [{type:'uint256',value:100}]
let transaction = await tronWeb.transactionBuilder.triggerSmartContract("419896f9376893d4882fa2375ab1732a943d19f8c2", "getTrxToTokenInputPrice(uint256)", {},parameter1,"412692a1d44cc5a51eefdb8e11e8cd1c20d802e474")
```

## justswap factory
```js
async getPool(){
  let contract = 'TXk8rQSAvPvBBNtqSoY6nCfsXWCSSpTVQF'
  let instance = await tronWeb.contract().at(contract)
  let exchange = await instance.getExchange('TPZddNpQJHu8UtKPY1PYDBv2J5p5QpJ6XW').call()
  // let exchange2 = this.tronWeb2.address.fromHex(exchange)

  let poolcontract = 'TPt2a3GtKMY5972mWa2aL3KKVY6ScWX2G2'
  let PoolInstance = await tronWeb.contract().at(poolcontract)
  let balance = await PoolInstance.balanceOf('TPt2a3GtKMY5972mWa2aL3KKVY6ScWX2G2').call()
},
```

## 监听事件
事件event一般是智能合约中的日志，你每调用一个方法，它执行后会得到什么结果呢：是成功还是失败？这一般可以通过事件来得到确定的答案。虽然在send()方法中有个`shouldPollResponse设置为true`参数，但是得到它的结果要等待60秒，这时间有点太感人了！回到事件的方法，这估计是确定性的最好方案了。
```js
// 查询并过滤事件
async buy(){
    this.isLoading = true
    this.confirmFlag = false
    let instance = this.$store.state.TronC2cInstance
    let hash = await instance.buyerSubmit(this.conformUid).send({feeLimit:20_000_000})

    //过滤事件以确定上链成功
    let i = 0
    let checkConfirm = async ()=> {
    i++
    if(i>9){
        //循环10次
        clearInterval(timer)
    }
    //过滤事件，每3秒10条，10次
    let res = await this.tronWeb2.getEventResult(this.tsteemOtc, {eventName:'BuyerSubmit', size:10})
    let checkEvent = async ()=> {
        return new Promise(resolve => {
        for(let t = 0; t < res.length; t++ ){
            if(hash === res[t].transaction){
            clearInterval(timer)
            resolve('ok')
            }
        }
        if(i === 10){
            resolve('false')
        }
        })
    }
    let event = await checkEvent()
    if(event === 'ok'){
        this.$router.push({path: 'mybuyer/'+this.conformNum})
    }else{
        alert("出错啦！请刷新后重试！")
    }
    }

    //设置定时器以更新
    let timer = setInterval(checkConfirm, 3000)
    // 通过$once来监听定时器，在beforeDestroy钩子时被清除。
    this.$once('hook:beforeDestroy', () => {
      clearInterval(timer)
    })
}

//watch监听事件，此方法不推荐！
async function triggercontract(){
    try {
        let instance = await tronWeb.contract().at('TQQg4EL8o1BSeKJY4MJ8TB8XK7xufxFBvK');
      
        instance.Transfer().watch((err, eventResult) => {
            if (err) {
                return console.error('Error with "method" event:', err);
            }
            if (eventResult) { 
                console.log('eventResult:',eventResult);
            }
          });

        let res = await instance.transfer('TWbcHNCYzqAGbrQteKnseKJdxfzBHyTfuh',500).send({
            feeLimit:100_000_000,
            callValue:0,
            shouldPollResponse:true
        });
        console.log(res);

    } catch (error) {
        console.log(error);
    }
}
triggercontract();
//这是tron的手册给出的方法。这方法是简单好用，但是watch的方法却是时灵时不灵的！OMG，这对于需要确定性结果的区块链简直是灾难啊！
```