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

# Installation
1.Add the Maven Central repository to your root build.gradle:
```groovy
allprojects {
    repositories {
        mavenCentral()
    }
}
```

2.Add the dependency to your app's build.gradle:
```groovy
dependencies {
	implementation 'io.gitee.ganlinux:connectsdk-android:1.0.0'
}
```
3.Configure AndroidManifest.xml (if needed):
```xml
<application
        tools:replace="android:allowBackup">
</application>
```

4.(Optional) Add Deeplink or Universal Link support:
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

## Version History

### 1.0.0 (Latest)
- Basic wallet connection support
- Ethereum chain support and Solana chain support
- Improved connection stability

# Quick Start
## Initialize SDK
OKXConnectSDK is a singleton object, when you create this, you can use it to call other APIs.

```java
val dAppInfo = DAppInfo(
        url = "connect.oker.vip",  // Your DApp's URL
        name = "OKX Connect Android Demo", // Display name
        icon = "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png", // App icon URL
        origin = "https://connect.oker.vip" // Origin URL for authentication
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
if(okxConnect.connectionState.value == OKConnectionState.SUSPENDED){
       // Handle suspended state
}
```
You can get the connection state types in class "OKConnectionState".

**Get Connection State**

```java
if(okxConnect.isConnected()){
       ...
}
```
Use this to check if connected.

**Connect to Wallet**

```java
val selectedMethods = listOf(RequestConnectAndSignMethod(EthMethod.PersonalSign("0x4d7920656d61696c206973206a6f686e40646f652e636f6d202d2031373237353937343537313336"), ETH))
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
val connectParams = ConnectParams(requiredNamespaces = requiredNamespaces, optionalNamespaces = optionalNamespaces, connectRequestMethods = selectedMethods, sessionConfig = sessionConfig)

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
if(okxConnect.connectionState.value == OKConnectionState.SUSPENDED){
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


## Ethereum Networks
| Network | Chain ID | Constant |
|---------|----------|----------|
| Ethereum | eip155:1 | Ethereum.CHAIN_ID.ETH |
| Polygon | eip155:137 | Ethereum.CHAIN_ID.POLYGON |
| Binance Smart Chain | eip155:56 | Ethereum.CHAIN_ID.BNB |
| Fuse | eip155:122 | Ethereum.CHAIN_ID.FUSE |
| Ronin | eip155:2020 | Ethereum.CHAIN_ID.RONIN |
| Harmony | eip155:1666600000 | Ethereum.CHAIN_ID.HARMONY |

## Solana Networks
| Network | Chain ID | Constant |
|---------|----------|----------|
| Solana Mainnet | solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.SOLANA_MAINNET |
| Soon Mainnet | soon:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.SOON_MAINNET |
| Soon Testnet | soon:4uhcVJyU9pJkvQyS88uRDiswHXSCkY3z | Solana.CHAIN_ID.SOON_TESTNET |
| Eclipse Mainnet | eclipse:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.ECLIPSE_MAINNET |






