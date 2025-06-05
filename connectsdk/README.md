# OKX Connect Android SDK
![Platform](https://img.shields.io/badge/platform-android-green.svg)
![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

一个功能强大的Wallet Connect SDK，用于将 OKX Connect 功能集成到您的 Android 应用程序中。

## 主要特性
- 🔒 安全的钱包连接和认证
- 🔄 多网络支持 (EVM系和Solana系)
- 📱 支持自定义 deeplink 和 universal link
- 🌐 可自定义 RPC 节点
- ⚡ 高效的连接状态管理
- 💫 自动恢复连接

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
  - [默认网络管理](#默认网络管理)
- [网络相关支持](#网络相关支持)
  - [支持连接时签名](#支持连接时签名)
  - [支持EVM系RPC请求](#支持EVM系RPC请求)
  - [EVM 网络](#evm-网络)
    - [EVM 方法](#evm-方法)
  - [Solana 网络](#solana-网络)
    - [Solana 方法](#solana-方法)
- [版本历史](#版本历史)

# 系统要求
- Android SDK 23 或更高版本
- Java 11

# 获取集成授权
请向我们的 BD 同事提供您的官方网站域名、应用程序包名和签名指纹信息。

# 快速开始
## 1. 在项目中集成SDK
我们提供两种集成方式：aar集成和Maven仓库集成，接入者可根据自身项目情况使用对应的接入方式。
### 1.1 添加 aar 文件
将 `okx-connectsdk-1.0.0.aar`和`okx-connectsdk-android-1.0.0.aar` 复制到项目的 `app/libs` 目录

### 1.2 配置 build.gradle
在项目级 build.gradle 添加：
```groovy
allprojects {
    repositories {
        // ... 其他仓库
        flatDir {
            dirs 'libs'
        }
    }
}
```

在 app 模块的 build.gradle 添加：
```groovy
dependencies {
    //SDK依赖
    implementation(name: 'okx-connectsdk-1.0.0', ext: 'aar')
    implementation(name: 'okx-connectsdk-android-1.0.0', ext: 'aar')
    
    // 其他依赖
    implementation 'com.squareup.okhttp3:okhttp:4.12.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.12.0'
    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3'
    implementation 'androidx.security:security-crypto:1.1.0-alpha06'
    implementation 'org.bouncycastle:bcprov-jdk15on:1.70'
    implementation 'com.github.joshjdevl.libsodiumjni:libsodium-jni-aar:2.0.2'
    implementation 'androidx.lifecycle:lifecycle-process:2.6.1'
}
```

## 2. 配置 AndroidManifest.xml（如需要）：
```xml
<application
    tools:replace="android:allowBackup">
</application>
```

## 3. （可选）添加 Deeplink 或 Universal Link 支持：
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


## 4. 初始化 SDK
创建后您将得到一个OKXConnectSDK类型的单例对象，您可以使用它来调用其他 API。

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
## 5. 连接管理

**连接状态监听**

您可以使用此方法来监听连接状态变化:
```java
okxConnect.connectionState.collect { state ->
        when (state) {
  OKConnectionState.CONNECTED -> {
    // 处理连接成功状态
  }
  OKConnectionState.DISCONNECTED -> {
    // 处理断开连接状态
  }
  OKConnectionState.CONNECTING -> {
    // 处理正在连接状态
  }
    else -> {
    // 处理其他状态...
  }
}
}
```

连接状态类型介绍
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
用于检查是否有钱包连接。

**连接到钱包**

```java
val connectRequestMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("Hello World!"), ETH))
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
此API 返回值为此连接任务对象，您可以自行取消这个任务。
成功回调中包含SessionInfo（会话信息）和ConnectRequestMethodResponse（请求方法返回内容）。
失败回调返回值为OKXConnectException类型。

**重要提示：** 如果`requiredNamespaces`中有钱包不支持的链，连接将直接失败。如果`optionalNamespaces`中有钱包不支持的链，将被忽略。

**断开连接**

断开已连接钱包，并删除当前会话，如果要切换连接钱包，请先断开当前钱包。
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
回调中返回连接的信息(SessionInfo)和状态(OKConnectionState)。

**暂停和恢复连接**
```java
okxConnect.suspendConnection()  //暂停
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
        okxConnect.resumeConnection()  //恢复
}
```
- suspendConnection - 用来临时关闭WebSocket，以减少网络资源占用。一般用于在特定的 UI 页面，不需要连接或交易请求的场景，可以手动控制WebSocket连接。
- resumeConnection - 重新建立与Websocket服务器的连接。


## 6. 发送请求

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
您可以使用方法实体来发起请求，这个方法将返回请求任务对象，您可以自行取消这个任务。您可以在 EthMethod 和 SolanaMethod 中找到支持的方法的定义，在 EthMethodResponse 和 SolanaMethodResponse 中找到响应类型的定义。

## 默认网络管理
**设置默认网络和 RPC URL**

在连接多个网络的状况下，如果没有明确指定当前操作所在网络，则通过默认网络进行交互。
```java
okxConnect.setDefaultChain("eip155:137", "https://polygon.drpc.org")
```

**获取默认网络**

获取您之前设置的默认网络。
```java
okxConnect.getDefaultChain("solana")
```

## 7. 支持连接时签名
可以让您在连接钱包时同时进行签名操作，避免多次操作的繁琐体验。
```java
//设置请求方法
//ETH PersonalSign
val connectRequestMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("Hello, World!"), ETH))
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
连接成功后，可以通过 ConnectRequestMethodResponse.Success 类型的result字段获取签名信息。

## 8. 支持EVM系RPC请求
如果当前EVM系的方法无法满足需求时，可通过配置 RPC 实现更多功能。

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
成功后的回调类型为EvmRpcResponse，可通过其内部JsonElement类型参数获取RPC返回内容。

## 8. EVM 网络
### EVM 方法
**添加自定义网络**

如果钱包无某个EVM网络，可通过此方法添加。
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
成功后的回调类型为WalletAddEthereumChain类型，参数caipAccount为钱包在此网络的地址。

**切换网络**
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
此方法用来切换到指定 chainId 的网络上。

**添加代币**
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
让OKX钱包上会显示特定代币的资产。成功后的回调类型为WalletWatchAsset，其中的isAdded参数用来判断是否添加成功。

**连接账户**
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
成功后的回调类型为RequestAccounts，访问其参数可以得到用户帐户地址。

**获取 chainId**
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
成功后的回调类型为ChainId，访问其参数chainId，获取用户当前的链 ID。

**签署消息**
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
成功后的回调类型为PersonalSign，访问其参数signature，获取签名结果。

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
成功后的回调类型为SignTypedDataV4，访问其参数signature，获取签名结果。

**签名交易**
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
成功后的回调类型为SendTransaction，访问其参数txHash，获取交易hash值。

**部分EVM网络常量定义**

| 网络 | 链 ID | 常量                  |
|---------|----------|---------------------------|
| Ethereum | eip155:1 | Ethereum.CHAIN_ID.ETH     |
| Polygon | eip155:137 | Ethereum.CHAIN_ID.POLYGON |
| Binance Smart Chain | eip155:56 | Ethereum.CHAIN_ID.BSC     |

## Solana 网络
### Solana 方法

**签名信息**
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
成功后的回调类型为SignMessage，访问其参数signature，获取签名结果。

**签名交易（不发送）**
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
成功后的回调类型为SignTransaction，访问其参数signature，获取签名结果。

**批量签署交易**
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
成功后的回调类型为SignAllTransactions，访问其参数transactions，获取签名结果列表。

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
    val from = response.from
    //do something
  }
}
```
成功后的回调类型为SignAndSendTransaction，访问其参数获取结果。

**部分Solana网络常量定义**

| 网络 | 链ID                                      | 常量 |
|---------|------------------------------------------|----------|
| Solana Mainnet | solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp  | Solana.CHAIN_ID.SOLANA_MAINNET |
| Soon Mainnet | soon:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp    | Solana.CHAIN_ID.SOON_MAINNET |
| Soon Testnet | soon:4uhcVJyU9pJkvQyS88uRDiswHXSCkY3z    | Solana.CHAIN_ID.SOON_TESTNET |
| Eclipse Mainnet | eclipse:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.ECLIPSE_MAINNET |

## 版本历史
### 1.0.0（最新）
- 钱包连接支持
- 支持以太坊系列网络和Solana系列网络
- 改进连接稳定性


