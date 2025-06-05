# OKX Connect Android SDK
![Platform](https://img.shields.io/badge/platform-android-green.svg)
![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

A powerful SDK for integrating OKX Connect functionality into your Android applications.

## Key Features
- 🔒 Secure wallet connection and authentication
- 🔄 Multi-chain support (Ethereum, Solana)
- 📱 Deep Link and Universal Link support
- 🌐 Customizable RPC endpoints
- ⚡ Efficient connection state management
- 💫 Automatic session restoration

## Table of Contents
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Features](#core-features)
    - [Connection Management](#connection-management)
    - [Send Request](#send-request)
    - [Default Chain Management](#default-chain-management)
- [Blockchain Support](#blockchain-support)
    - [Ethereum](#ethereum)
    - [Solana](#solana)
  
# Requirements
- Android SDK 23 or higher
- Java 11

# Obtain Integration Authorization
Provide your official website domain, application package name and fingerprint to our BD colleagues. 

# Installation
1. Add the Maven Central repository to your root build.gradle:
```groovy
allprojects {
    repositories {
        mavenCentral()
    }
}
```

2. Add the dependency to your app's build.gradle:
```groovy
dependencies {
	implementation 'io.gitee.ganlinux:connectsdk-android:1.0.0'
}
```
3. Configure AndroidManifest.xml (if needed):
```xml
<application
        tools:replace="android:allowBackup">
</application>
```

4. (Optional) Add Deeplink or Universal Link support:
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

# Example
For a complete implementation example, check out our [Demo App](https://github.com/okx/connectsdk-android-demo).


# Quick Start
## Initialize SDK
OKXConnectSDK is a singleton object, when you create this, you can use it to call other APIs.

```java
val dAppInfo = DAppInfo(
        url = "https://connect.okx.com",  // Your DApp's URL
        name = "OKX Connect Android Demo", // Display name
        icon = "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png" // App icon URL
)

val okxConnect = OKXConnectSDKAndroid.create(
    context = this,
    dappInfo = dAppInfo,
    onSuccess = {
        // SDK initialized successfully
        // Ready to connect to wallet
    },
    onError = { error ->
       // Handle initialization errors
       // e.g., invalid configuration, network issues
    }
)
```
# Core Features
## Connection Management

**Connection observe**

```java
val connectionState by okxConnect.connectionState.collectAsState()
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
       // Handle suspended state
}
```
You can get the connection state types in class "OKConnectionState".

**Get Connection State**

```java
if (okxConnect.isConnected()) {
       ...
}
```
Use this to check if connected.

**Connect to Wallet**

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
     // callback when connection is successful
     },
     onError = { error ->
     // callback when connection fails
     }
)
```
This example use ethereum chain. You can define the chains you want.
The API returns the connect job, you can cancel this job by yourself.

**IMPORTANT:** If there are chains not support in `requiredNamespaces`, it will connect failed. If there are chains not support in `optionalNamespaces`, it will ignore it.

**Disconnect**

```java
okxConnect.disconnect()
```

**ConnectionState Listener**
```java
private val connectionStateListener = object : ConnectionStateListener {
    override fun onConnectionStateChange(state: OKConnectionState, session: SessionInfo?) {
		...
    }
}
okxConnect.addConnectionStateListener(connectionStateListener)
// remove the listener when not needed    
okxConnect.removeConnectionStateListener(connectionStateListener)
```
It will return the information(SessionInfo) and state(OKConnectionState) about the connection.

**Restore Connection**
```java
okxConnect.restoreSessionIfExit()
```
Restore previous connection before displaying connection information.

**Suspend and resume connection**
```java
okxConnect.suspendConnection()  //suspend
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
    okxConnect.resumeConnection()  //resume
}
```
- suspendConnection - Temporarily suspends the connection by closing WebSocket to reduce network usage.Used when connection or transaction requests are not needed, such as in specific UI pages.
- resumeConnection - Reestablishes the connection with the websocket server.


## Send Request

```java
    val request = RequestParamsMethod(method = EthMethod.PersonalSign(EVM_NORMAL_SIGNDATA), chainId = ETH)
    okxConnect.request(request){ result ->
	    ...
    }
```
You can use the method entity to start request, this method will return the request job and cancel this job by yourself.You can find the support methods in EthMethod and SolanaMethod, find the response type in EthMethodResponse and SolanaMethodResponse. 


## Default Chain Management 
**Set selected chain and RPC url**
```java
okxConnect.setDefaultChain("eip155:137", "https://polygon.drpc.org")
```

**Get selected chain**
```java
okxConnect.getDefaultChain("solana")
```
Get the selected chain what you set before.


# Blockchain Support
## Features
- Connect and sign support
- HTTP RPC support
- EVM and SVM series chains support

## Connect and sign support
```java
//set request method
//ETH PersonalSign
val connectRequestMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("0x4d7920656d61696c206973206a6f686e40646f652e636f6d202d2031373237353937343537313336"), ETH))
//Solana SignBase58Message
//val connectRequestMethods = listOf(RequestConnectAndSignMethod(SolanaMethod.SignBase58Message("xx"), SOLANA_MAINNET))
val requiredNamespaces = listOf(
    Request.RequestAccounts.Namespace(
      namespace = NAMESPACE_EVM,
      chains = listOf(ETH, POLYGON),
      rpcMap = mapOf(POLYGON to "https://polygon.drpc.org")
    ))
val sessionConfig = SessionConfig(redirect = "okxconnect://demo")
val connectParams = ConnectParams(requiredNamespaces = requiredNamespaces, connectRequestMethods = connectRequestMethods, sessionConfig = sessionConfig)
val connectJob = okxConnect.connect(
  connectParams = connectParams,
  onSuccess = { sessionInfo, methodResults ->
    // callback when connection is successful
      val response = methodResults?.find {
        it.chainId == ETH && it.method == PERSONAL_SIGN }
        if (response != null && response is Response.Accounts.ConnectRequestMethodResponse.Success) {
          val signResult = response.result
          //check
        }
   },
   onError = { error ->
    // callback when connection fails
    }
)
```

## HTTP RPC support
**Evm RPC Method**
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

## EVM Networks
| Network | Chain ID | Constant                  |
|---------|----------|---------------------------|
| Ethereum | eip155:1 | Ethereum.CHAIN_ID.ETH     |
| Polygon | eip155:137 | Ethereum.CHAIN_ID.POLYGON |
| Binance Smart Chain | eip155:56 | Ethereum.CHAIN_ID.BSC     |


### EVM Methods
**Add Custom Chain**
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

**Switch Chain**
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

**Watch Asset**
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
**Request Accounts**
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

**ChainId**
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

**PersonalSign**
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
**SignTypedDataV4**
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
**SendTransaction**
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

## Solana Networks
| Network | Chain ID | Constant |
|---------|----------|----------|
| Solana Mainnet | solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.SOLANA_MAINNET |
| Soon Mainnet | soon:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.SOON_MAINNET |
| Soon Testnet | soon:4uhcVJyU9pJkvQyS88uRDiswHXSCkY3z | Solana.CHAIN_ID.SOON_TESTNET |
| Eclipse Mainnet | eclipse:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.ECLIPSE_MAINNET |

### Solana Methods

**SignMessage**
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

**SignTransaction**
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

**SignAllTransactions**
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

**SignAndSendTransaction**
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

## Version History

### 1.0.0 (Latest)
- Basic wallet connection support
- Ethereum chain support and Solana chain support
- Improved connection stability


