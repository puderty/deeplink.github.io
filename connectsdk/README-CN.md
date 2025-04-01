# OKX Connect Android SDK
![Platform](https://img.shields.io/badge/platform-android-green.svg)
![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

一个功能强大的Wallet Connect SDK，用于将 OKX Connect 功能集成到您的 Android 应用程序中。

## 主要特性
- 🔒 安全的钱包连接和认证
- 🔄 多链支持 (EVM系和Solana系)
- 📱 支持自定义 Deep Link 和 Universal Link
- 🌐 可自定义 RPC 节点
- ⚡ 高效的连接状态管理
- 💫 自动会话恢复

## 目录
## 目录
- [系统要求](#系统要求)
- [获取集成授权](#获取集成授权)
- [安装](#安装)
- [示例](#示例)
- [快速开始](#快速开始)
  - [初始化 SDK](#初始化-sdk)
- [核心功能](#核心功能)
  - [连接管理](#连接管理)
  - [发送请求](#发送请求)
  - [默认链管理](#默认链管理)
- [链相关支持](#链相关支持)
  - [特色功能](#特色功能)
  - [支持连接和签名](#支持连接和签名)
  - [支持 HTTP RPC](#支持-http-rpc)
  - [EVM 网络](#evm-网络)
    - [EVM 方法](#evm-方法)
  - [Solana 网络](#solana-网络)
    - [Solana 方法](#solana-方法)
- [版本历史](#版本历史)
  
# 系统要求
- Android SDK 23 或更高版本
- Java 11

# 获取集成授权
请向我们的 BD 同事提供您的官方网站域名、应用程序包名和指纹信息。

# 安装
1. 在根目录的 build.gradle 中添加 Maven Central 仓库：
```groovy
allprojects {
    repositories {
        mavenCentral()
    }
}
```

2. 在应用的 build.gradle 中添加依赖：
```groovy
dependencies {
  implementation 'io.gitee.ganlinux:connectsdk-android:1.0.0'
}
```
3. 配置 AndroidManifest.xml（如需要）：
```xml
<application
    tools:replace="android:allowBackup">
</application>
```

4. （可选）添加 Deeplink 或 Universal Link 支持：
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />

    <data
        android:scheme="okxconnect"
        android:host="demo" />

</intent-filter>

<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />

    <data
        android:scheme="https"
        android:host="connectsdk.com"
        android:pathPrefix="/demo" />
</intent-filter>
```

# 示例
完整的实现示例，请查看我们的 [演示应用](https://github.com/okx/connectsdk-android-demo).


# 快速开始
## 初始化 SDK
OKXConnectSDK 是一个单例对象，创建后您可以使用它来调用其他 API。

```java
val dAppInfo = DAppInfo(
  url = "https://connect.okx.com",  // 您的 DApp URL
  name = "OKX Connect Android Demo", // 显示名称
  icon = "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png" // 应用图标 URL
)

val okxConnect = OKXConnectSDKAndroid.create(
  context = this,
  dappInfo = dAppInfo,
  onSuccess = {
      // SDK 初始化成功
      // 准备连接钱包
  },
  onError = { error ->
     // 处理初始化错误
     // 例如：配置无效、网络问题
  }
)
```
# 核心功能
## 连接管理

**连接状态观察**

```java
val connectionState by okxConnect.connectionState.collectAsState()
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
       // 处理暂停状态
}
```

您可以在 OKConnectionState 类中找到所有连接状态类型。

OKConnectionState详解
```java
enum class OKConnectionState(val state: Int) {
  CONNECTED(0), //已连接
  DISCONNECTED(1), //已断开连接
  CONNECTING(2), //连接中
  RECONNECTING(3), //重新连接中
  CONNECT_FAILURE(4), //连接失败
  RECONNECT_FAILURE(5), //重新连接失败
  SUSPENDED(6); //暂停连接
}
```

**获取连接状态**

```java
if (okxConnect.isConnected()) {
       ...
}
```
用于检查是否已连接。

**连接到钱包**

```java
val connectRequestMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("0x4d7920656d61696c206973206a6f686e40646f652e636f6d202d2031373237353937343537313336"), ETH))
val requiredNamespaces = listOf(Request.RequestAccounts.Namespace(
	namespace = NAMESPACE_EVM,
	chains = listOf(ETH, POLYGON),
        rpcMap = mapOf(POLYGON to "https://polygon.drpc.org")
))
val optionalNamespaces = listOf(Request.RequestAccounts.Namespace(
     namespace = NAMESPACE_EVM,
     chains = listOf(ETH)
))
val sessionConfig = SessionConfig(redirect = "okxconnect://demo")
val connectParams = ConnectParams(requiredNamespaces = requiredNamespaces, optionalNamespaces = optionalNamespaces, connectRequestMethods = connectRequestMethods, sessionConfig = sessionConfig)

val connectJob = okxConnect.connect(
     connectParams = connectParams,
     onSuccess = { sessionInfo, methodResults ->
     // 连接成功的回调
     },
     onError = { error ->
     // 连接失败的回调
     }
)
```
这个例子使用了以太坊链。您可以定义您想要的链。
此API 返回连接任务，您可以自行取消这个任务。

**重要提示：** 如果`requiredNamespaces`中有钱包不支持的链，连接将直接失败。如果`optionalNamespaces`中有钱包不支持的链，将被忽略。

**断开连接**

```java
okxConnect.disconnect()
```

**连接状态监听器**
```java
private val connectionStateListener = object : ConnectionStateListener {
    override fun onConnectionStateChange(state: OKConnectionState, session: SessionInfo?) {
		...
    }
}
okxConnect.addConnectionStateListener(connectionStateListener)
// 不需要时移除监听器    
okxConnect.removeConnectionStateListener(connectionStateListener)
```
它将返回关于连接的信息(SessionInfo)和状态(OKConnectionState)。

**暂停和恢复连接**
```java
okxConnect.suspendConnection()  //暂停
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
    okxConnect.resumeConnection()  //恢复
}
```
- suspendConnection - 通过关闭WebSocket，临时暂停连接以减少网络占用。在不需要连接或交易请求时使用，比如在特定的 UI 页面中。
- resumeConnection - 重新建立与Websocket服务器的连接。


## 发送请求

```java
val method = EthMethod.PersonalSign("Hello, World!")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.PersonalSign) {
        val signature = response.signature
        //do something
    }
}
```
您可以使用方法实体来发起请求，这个方法将返回请求任务，您可以自行取消这个任务。您可以在 EthMethod 和 SolanaMethod 中找到支持的方法，在 EthMethodResponse 和 SolanaMethodResponse 中找到响应类型。

## 默认链管理
**设置选定的链和 RPC URL**
```java
okxConnect.setDefaultChain("eip155:137", "https://polygon.drpc.org")
```

**获取选定的链**
```java
okxConnect.getDefaultChain("solana")
```
获取您之前设置的选定链。


# 链相关支持
## 特色功能
- 支持连接和签名
- 支持 HTTP RPC
- 支持 EVM 和 Solana 系列链

## 支持连接和签名
```java
//设置请求方法
//ETH PersonalSign
val connectRequestMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("0x4d7920656d61696c206973206a6f686e40646f652e636f6d202d2031373237353937343537313336"), ETH))
//Solana SignBase58Message
//val connectRequestMethods = listOf(RequestConnectAndSignMethod(SolanaMethod.SignBase58Message("xx"), SOLANA_MAINNET))
val requiredNamespaces = listOf(
    Request.RequestAccounts.Namespace(
      namespace = NAMESPACE_EVM,
      chains = listOf(ETH, POLYGON),
      rpcMap = mapOf(POLYGON to "https://polygon.drpc.org")
    )
)
val sessionConfig = SessionConfig(redirect = "okxconnect://demo")
val connectParams = ConnectParams(requiredNamespaces = requiredNamespaces, connectRequestMethods = connectRequestMethods, sessionConfig = sessionConfig)
val connectJob = okxConnect.connect(
  connectParams = connectParams,
  onSuccess = { sessionInfo, methodResults ->
    // 连接成功的回调
      val response = methodResults?.find {
        it.chainId == ETH && it.method == PERSONAL_SIGN }
        if (response != null && response is Response.Accounts.ConnectRequestMethodResponse.Success) {
          val signResult = response.result
          //检查
        }
   },
   onError = { error ->
      // 连接失败的回调
   }
)
```

## 支持 HTTP RPC
**Evm RPC 方法**
```java
val methodName = "eth_getTransactionByHash"
val params = buildJsonArray { add("0xd62fa4ea3cf7ee3bf6f5302b764490730186ed6a567c283517e8cb3c36142e1a") }
val method = EthMethod.EvmRPCMethod(methodName, params)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
  val response = result.getOrNull()
  if (response is EthMethodResponse.EvmRpcResponse) {
    val rpcResult = response.result
    //do something
  }
}
```

## EVM 网络
| 网络 | 链 ID | 常量                  |
|---------|----------|---------------------------|
| Ethereum | eip155:1 | Ethereum.CHAIN_ID.ETH     |
| Polygon | eip155:137 | Ethereum.CHAIN_ID.POLYGON |
| Binance Smart Chain | eip155:56 | Ethereum.CHAIN_ID.BSC     |


### EVM 方法
**添加自定义链**
```java
val method = EthMethod.WalletAddEthereumChain(
  listOf("https://explorer.fuse.io"), "0x7a", "Fuse",
  EthMethod.WalletAddEthereumChain.NativeCurrency("Fuse", "FUSE", 18), listOf("https://rpc.fuse.io")
)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.WalletAddEthereumChain) {
        val caipAccount = response.caipAccount
        //do something
    }
}
```

**链切换**
```java
val method = EthMethod.WalletSwitchEthereumChain(chainId = "0x7a")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.WalletSwitchEthereumChain) {
        //do something
    }
}
```

**观察资产**
```java
val options = AssetOptions("0xe0f63a424a4439cbe457d80e4f4b51ad25b2c56c", "SPX6900", "https://assets.coingecko.com/coins/images/31401/standard/sticker_%281%29.jpg?1702371083", 8)
val method = EthMethod.WalletWatchAsset("ERC20", options)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.WalletWatchAsset) {
        val addResult = response.isAdded
        //do something
    }
}
```
**请求账户信息**
```java
val method = EthMethod.RequestAccounts(emptyList(), 0L)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.RequestAccounts) {
        val accounts = response.accounts
        //do something
    }
}
```

**获取链Id**
```java
val method = EthMethod.ChainId()
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.ChainId) {
        val chainId = response.chainId
        //do something
    }
}
```

**个人签名**
```java
val method = EthMethod.PersonalSign("Hello, World!")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.PersonalSign) {
        val signature = response.signature
        //do something
    }
}
```
**SignTypedDataV4签名方法**
```java
private val TYPEDDATAV_JSONSTRING = buildJsonObject {
  putJsonObject("message") {
  putJsonObject("from") {
    put("name", "Cow")
    put("wallet", "0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826")
  }
  putJsonObject("to") {
    put("name", "Bob")
    put("wallet", "0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB")
  }
  put("contents", "Hello, Bob!")
  }
}.toString()
val method = EthMethod.SignTypedDataV4(TYPEDDATAV_JSONSTRING)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.SignTypedDataV4) {
        val signature = response.signature
        //do something
    }
}
```
**发送交易**
```java
val method = EthMethod.SendTransaction(gas = "0x2665f", from = "0xf2F3e73be57031114dd1f4E75c1DD87658be7F0E", to = "0xf2614A233c7C3e7f08b1F887Ba133a13f1eb2c55", value = "0x38d7ea4c68000", data = "0x2646478b000000000000000000000000eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee00000000000000000000000000000000000000000000000000038d7ea4c68000000000000000000000000000620fd5fa44be6af63715ef4e65ddfa0387ad13f5000000000000000000000000000000000000000000000000000000000000001b000000000000000000000000f2f3e73be57031114dd1f4e75c1dd87658be7f0e00000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000000700301ffff0201602352A9Eb5234466Eac23E59e7B99bCaE79C39c0BE9e53fd7EDaC9F859882AfdDa116645287C629040BE9e53fd7EDaC9F859882AfdDa116645287C62900602352A9Eb5234466Eac23E59e7B99bCaE79C39c01f2F3e73be57031114dd1f4E75c1DD87658be7F0E000bb800000000000000000000000000000000")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is EthMethodResponse.SendTransaction) {
        val txHash = response.txHash
        //do something
    }
}
```

## Solana 网络
| 网络 | 链ID                                      | 常量 |
|---------|------------------------------------------|----------|
| Solana Mainnet | solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp  | Solana.CHAIN_ID.SOLANA_MAINNET |
| Soon Mainnet | soon:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp    | Solana.CHAIN_ID.SOON_MAINNET |
| Soon Testnet | soon:4uhcVJyU9pJkvQyS88uRDiswHXSCkY3z    | Solana.CHAIN_ID.SOON_TESTNET |
| Eclipse Mainnet | eclipse:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.ECLIPSE_MAINNET |

### Solana 方法

**签名消息**
```java
val messageBytes = "Hello Solana".toByteArray(Charset.forName("UTF-8"))
val method = SolanaMethod.SignMessage(Base58.encode(messageBytes))
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is SolanaMethodResponse.SignMessage) {
        val txHash = response.signature
        //do something
    }
}
```

**签名交易**
```java
private val base58 = "transaction data"
private val address = "wallet address"
val method = SolanaMethod.SignTransaction(base58, address)
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is SolanaMethodResponse.SignTransaction) {
        val txHash = response.signature
        //do something
    }
}
```

**签名所有交易**
```java
private val base58 = "transaction data"
private val address = "wallet address"
val method = SolanaMethod.SignAllTransactions(listOf(SignTransaction(base58, address)))
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is SolanaMethodResponse.SignAllTransactions) {
        val transactions = response.transactions
        //do something
    }
}
```

**签名并发送交易**
```java
private val base58 = "transaction data"
private val address = "wallet address"
val method = SolanaMethod.SignAndSendTransaction(base58, address)
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request){ result ->
    val response = result.getOrNull()
    if (response is SolanaMethodResponse.SignAndSendTransaction) {
        val data = response.data
        val data = response.data
        //do something
    }
}
```

## 版本历史

### 1.0.0（最新）
- 基础钱包连接支持
- 支持以太坊链和Solana系列链
- 改进连接稳定性


