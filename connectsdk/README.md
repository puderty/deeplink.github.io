

# OKX Connect Android SDK 技术文档

## 1. 概述

**OKX Connect Android SDK** 是一个功能强大的 Wallet Connect 工具包，用于将 OKX 钱包连接功能集成到 Android 应用程序中，支持安全认证、多链交互和高效连接管理。本文档详细介绍了 SDK 的功能、集成步骤、核心 API 以及支持的网络和方法，旨在帮助开发者快速完成集成并实现去中心化应用（DApp）与 OKX 钱包的交互。

### 1.1 主要特性
- **安全连接**：通过加密协议实现钱包的认证和连接。
- **多网络支持**：兼容 EVM 系（如 Ethereum、Polygon）和 Solana 系网络。
- **灵活链接**：支持自定义 Deep Link 和 Universal Link。
- **自定义 RPC**：允许配置专属 RPC 节点，提升网络交互效率。
- **连接管理**：提供高效的连接状态管理和自动恢复机制。
- **签名优化**：支持连接时签名，简化用户操作。

### 1.2 版本信息
- **平台**：Android
- **版本**：1.0.0
- **许可**：MIT

## 2. 系统要求

- **Android SDK**：API 23（Android 6.0）或更高版本。
- **Java**：JDK 11 或以上。
- **开发环境**：Android Studio 2022.3.1 或更高版本推荐。

## 3. 获取集成授权

在集成 SDK 前，需向 OKX 商务团队（BD）申请授权。请提供以下信息：
- 官方网站域名（如 `https://your-dapp.com`）。
- 应用程序包名（如 `com.example.dapp`）。
- 签名指纹（SHA256，指通过 `keytool` 获取）。

联系方式：请通过 OKX 官网（`www.okx.com`）提交申请。

## 4. 集成步骤

### 4.1 添加 SDK 文件
1. 将以下 AAR 文件复制到项目 `app/libs` 目录：
   - `okx-connectsdk-1.0.0.aar`
   - `okx-connectsdk-android-1.0.0.aar`
2. 确保 `libs` 目录存在，若无则手动创建。

### 4.2 配置 Gradle
在 **项目级** `build.gradle` 中添加本地仓库：

```groovy
allprojects {
    repositories {
        google()
        mavenCentral()
        flatDir {
            dirs 'libs'
        }
    }
}
```

在 **模块级** `app/build.gradle` 中添加依赖：

```groovy
dependencies {
    // OKX Connect SDK
    implementation(name: 'okx-connectsdk-1.0.0', ext: 'aar')
    implementation(name: 'okx-connectsdk-android-1.0.0', ext: 'aar')

    // 必要依赖
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

**注意**：
- 同步项目（`Sync Project with Gradle Files`）以确保依赖加载。
- 确保网络连接正常以下载外部依赖。

### 4.3 配置 AndroidManifest.xml
在 `AndroidManifest.xml` 中添加以下配置以支持备份和链接：

```xml
<application
    android:allowBackup="true"
    tools:replace="android:allowBackup">
    <!-- Deep Link 和 Universal Link（可选） -->
    <activity android:name=".YourActivity">
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
    </activity>
</application>
```

**注意**：
- 替换 `YourActivity` 为实际接收 Deep Link 的 Activity。
- 根据 DApp 需求调整 `scheme` 和 `host`。

## 5. 初始化 SDK

初始化 SDK 是使用 OKX Connect 的第一步，生成单例对象以调用后续 API。

### 5.1 初始化代码

```kotlin
val dAppInfo = DAppInfo(
    url = "https://connect.okx.com", // DApp 官方 URL
    name = "OKX Connect Android Demo", // DApp 显示名称
    icon = "https://static.okx.com/cdn/assets/imgs/247/58E63FEA47A2B7D7.png" // DApp 图标 URL
)

val okxConnect = OKXConnectSDKAndroid.create(
    context = this,
    dappInfo = dAppInfo,
    onSuccess = {
        // 初始化成功，可开始连接钱包
        Log.d("OKXConnect", "SDK 初始化成功")
    },
    onError = { error ->
        // 处理初始化错误
        Log.e("OKXConnect", "初始化失败: ${error.message}")
    }
)
```

**参数说明**：
- `context`：应用上下文（如 Activity 或 Application）。
- `dAppInfo`：DApp 元数据，用于钱包显示。
- `onSuccess`：初始化成功回调。
- `onError`：初始化失败回调，返回 `OKXConnectException`。

**注意**：
- 确保 `url` 和 `icon` 为有效 HTTPS 地址。
- 初始化只需执行一次，通常在应用启动时。

## 6. 核心功能

### 6.1 连接管理

#### 6.1.1 连接状态监听
通过协程 Flow 监听连接状态变化：

```kotlin
okxConnect.connectionState.collect { state ->
    when (state) {
        OKConnectionState.CONNECTED -> Log.d("OKXConnect", "已连接")
        OKConnectionState.DISCONNECTED -> Log.d("OKXConnect", "已断开")
        OKConnectionState.CONNECTING -> Log.d("OKXConnect", "连接中")
        else -> Log.d("OKXConnect", "其他状态: $state")
    }
}
```

**连接状态枚举**：

```kotlin
enum class OKConnectionState(val state: Int) {
    CONNECTED(0),      // 已连接
    DISCONNECTED(1),   // 已断开
    CONNECTING(2),     // 连接中
    RECONNECTING(3),   // 重新连接中
    CONNECT_FAILURE(4),// 连接失败
    RECONNECT_FAILURE(5), // 重新连接失败
    SUSPENDED(6)       // 暂停连接
}
```

#### 6.1.2 检查连接状态

```kotlin
if (okxConnect.isConnected()) {
    Log.d("OKXConnect", "钱包已连接")
} else {
    Log.d("OKXConnect", "钱包未连接")
}
```

#### 6.1.3 连接钱包

```kotlin
val connectRequestMethods = listOf(
    RequestConnectAndSignMethod(EthMethod.PersonalSign("Hello World!"), ETH)
)
val requiredNamespaces = listOf(
    Request.RequestAccounts.Namespace(
        namespace = NAMESPACE_EVM,
        chains = listOf(ETH, POLYGON),
        rpcMap = mapOf(POLYGON to "https://polygon.drpc.org")
    )
)
val optionalNamespaces = listOf(
    Request.RequestAccounts.Namespace(
        namespace = NAMESPACE_EVM,
        chains = listOf(ETH)
    )
)
val sessionConfig = SessionConfig(redirect = "okxconnect://demo")
val connectParams = ConnectParams(
    requiredNamespaces = requiredNamespaces,
    optionalNamespaces = optionalNamespaces,
    connectRequestMethods = connectRequestMethods,
    sessionConfig = sessionConfig
)

val connectJob = okxConnect.connect(
    connectParams = connectParams,
    onSuccess = { sessionInfo, methodResults ->
        Log.d("OKXConnect", "连接成功: $sessionInfo")
    },
    onError = { error ->
        Log.e("OKXConnect", "连接失败: ${error.message}")
    }
)
```

**参数说明**：
- `requiredNamespaces`：必须支持的链（如 ETH、Polygon），不支持则连接失败。
- `optionalNamespaces`：可选链，不支持将被忽略。
- `connectRequestMethods`：连接时执行的签名方法。
- `sessionConfig`：会话配置，如 Deep Link 重定向 URI。

**注意**：
- `connectJob` 可用于取消连接任务。
- 确保钱包支持指定的链和方法。

#### 6.1.4 断开连接

```kotlin
okxConnect.disconnect()
Log.d("OKXConnect", "已断开钱包连接")
```

**注意**：断开后需重新连接以恢复会话。

#### 6.1.5 连接状态监听器

```kotlin
private val connectionStateListener = object : ConnectionStateListener {
    override fun onConnectionStateChange(state: OKConnectionState, session: SessionInfo?) {
        Log.d("OKXConnect", "状态变更: $state, 会话: $session")
    }
}
okxConnect.addConnectionStateListener(connectionStateListener)
// 移除监听器
okxConnect.removeConnectionStateListener(connectionStateListener)
```

#### 6.1.6 暂停和恢复连接

```kotlin
// 暂停 WebSocket 连接
okxConnect.suspendConnection()
Log.d("OKXConnect", "连接已暂停")

// 恢复连接
if (okxConnect.connectionState.value == OKConnectionState.SUSPENDED) {
    okxConnect.resumeConnection()
    Log.d("OKXConnect", "连接已恢复")
}
```

**场景**：暂停连接适合在非交互页面（如设置页）减少资源占用。

### 6.2 发送请求

发送链上请求（如签名、交易）：

```kotlin
val method = EthMethod.PersonalSign("Hello, World!")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.PersonalSign) {
            Log.d("OKXConnect", "签名结果: ${response.signature}")
        }
    } ?: Log.e("OKXConnect", "请求失败: ${result.exceptionOrNull()?.message}")
}
```

**支持方法**：
- EVM：`EthMethod`（如 `PersonalSign`、`SendTransaction`）。
- Solana：`SolanaMethod`（如 `SignMessage`、`SignTransaction`）。

**注意**：检查 `EthMethodResponse` 或 `SolanaMethodResponse` 以处理响应。

### 6.3 默认网络管理

#### 6.3.1 设置默认网络

```kotlin
okxConnect.setDefaultChain("eip155:137", "https://polygon.drpc.org")
Log.d("OKXConnect", "默认网络设置为 Polygon")
```

#### 6.3.2 获取默认网络

```kotlin
val defaultChain = okxConnect.getDefaultChain("solana")
Log.d("OKXConnect", "默认 Solana 网络: $defaultChain")
```

**注意**：默认网络用于未指定链 ID 的操作。

## 7. 网络支持

### 7.1 连接时签名

支持在连接时执行签名，优化用户体验：

```kotlin
val connectRequestMethods = listOf(
    RequestConnectAndSignMethod(EthMethod.PersonalSign("Hello, World!"), ETH)
)
val requiredNamespaces = listOf(
    Request.RequestAccounts.Namespace(
        namespace = NAMESPACE_EVM,
        chains = listOf(ETH, POLYGON),
        rpcMap = mapOf(POLYGON to "https://polygon.drpc.org")
    )
)
val sessionConfig = SessionConfig(redirect = "okxconnect://demo")
val connectParams = ConnectParams(
    requiredNamespaces = requiredNamespaces,
    connectRequestMethods = connectRequestMethods,
    sessionConfig = sessionConfig
)

okxConnect.connect(
    connectParams = connectParams,
    onSuccess = { sessionInfo, methodResults ->
        methodResults?.find { it.chainId == ETH && it.method == PERSONAL_SIGN }?.let { response ->
            if (response is Response.Accounts.ConnectRequestMethodResponse.Success) {
                Log.d("OKXConnect", "签名结果: ${response.result}")
            }
        }
    },
    onError = { error ->
        Log.e("OKXConnect", "连接失败: ${error.message}")
    }
)
```

### 7.2 EVM 网络支持

#### 7.2.1 EVM RPC 请求

支持自定义 RPC 调用：

```kotlin
val methodName = "eth_getTransactionByHash"
val params = buildJsonArray { add("0xd62fa4ea3cf7ee3bf6f5302b764490730186ed6a567c283517e8cb3c36142e1a") }
val method = EthMethod.EvmRPCMethod(methodName, params)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.EvmRpcResponse) {
            Log.d("OKXConnect", "RPC 结果: ${response.result}")
        }
    }
}
```

#### 7.2.2 EVM 方法

- **添加网络**：

```kotlin
val method = EthMethod.WalletAddEthereumChain(
    blockExplorerUrls = listOf("https://explorer.fuse.io"),
    chainId = "0x7a",
    chainName = "Fuse",
    nativeCurrency = EthMethod.WalletAddEthereumChain.NativeCurrency("Fuse", "FUSE", 18),
    rpcUrls = listOf("https://rpc.fuse.io")
)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.WalletAddEthereumChain) {
            Log.d("OKXConnect", "添加网络地址: ${response.caipAccount}")
        }
    }
}
```

- **切换网络**：

```kotlin
val method = EthMethod.WalletSwitchEthereumChain(chainId = "0x7a")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.WalletSwitchEthereumChain) {
            Log.d("OKXConnect", "网络切换成功")
        }
    }
}
```

- **添加代币**：

```kotlin
val options = AssetOptions(
    address = "0xe0f63a424a4439cbe457d80e4f4b51ad25b2c56c",
    symbol = "SPX6900",
    image = "https://assets.coingecko.com/coins/images/31401/standard/sticker_%281%29.jpg?1702371083",
    decimals = 8
)
val method = EthMethod.WalletWatchAsset("ERC20", options)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.WalletWatchAsset) {
            Log.d("OKXConnect", "代币添加: ${response.isAdded}")
        }
    }
}
```

- **连接账户**：

```kotlin
val method = EthMethod.RequestAccounts(emptyList(), 0L)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.RequestAccounts) {
            Log.d("OKXConnect", "账户: ${response.accounts}")
        }
    }
}
```

- **获取 Chain ID**：

```kotlin
val method = EthMethod.ChainId()
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.ChainId) {
            Log.d("OKXConnect", "Chain ID: ${response.chainId}")
        }
    }
}
```

- **签署消息**：

```kotlin
val method = EthMethod.PersonalSign("Hello, World!")
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.PersonalSign) {
            Log.d("OKXConnect", "签名: ${response.signature}")
        }
    }
}
```

- **SignTypedDataV4 签名**：

```kotlin
val typedData = buildJsonObject {
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
val method = EthMethod.SignTypedDataV4(typedData)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.SignTypedDataV4) {
            Log.d("OKXConnect", "签名: ${response.signature}")
        }
    }
}
```

- **签名交易**：

```kotlin
val method = EthMethod.SendTransaction(
    gas = "0x2665f",
    from = "0xf2F3e73be57031114dd1f4E75c1DD87658be7F0E",
    to = "0xf2614A233c7C3e7f08b1F887Ba133a13f1eb2c55",
    value = "0x38d7ea4c68000",
    data = "0x2646478b..."
)
val request = RequestParamsMethod(method = method, chainId = ETH)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is EthMethodResponse.SendTransaction) {
            Log.d("OKXConnect", "交易 Hash: ${response.txHash}")
        }
    }
}
```

#### 7.2.3 EVM 网络常量

| 网络                | 链 ID         | 常量                     |
|---------------------|---------------|--------------------------|
| Ethereum            | eip155:1      | Ethereum.CHAIN_ID.ETH    |
| Polygon             | eip155:137    | Ethereum.CHAIN_ID.POLYGON|
| Binance Smart Chain | eip155:56     | Ethereum.CHAIN_ID.BSC    |

### 7.3 Solana 网络支持

#### 7.3.1 Solana 方法

- **签名消息**：

```kotlin
val messageBytes = "Hello Solana".toByteArray(Charset.forName("UTF-8"))
val method = SolanaMethod.SignMessage(Base58.encode(messageBytes))
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is SolanaMethodResponse.SignMessage) {
            Log.d("OKXConnect", "签名: ${response.signature}")
        }
    }
}
```

- **签名交易（不发送）**：

```kotlin
val base58 = "transaction data"
val address = "wallet address"
val method = SolanaMethod.SignTransaction(base58, address)
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is SolanaMethodResponse.SignTransaction) {
            Log.d("OKXConnect", "签名: ${response.signature}")
        }
    }
}
```

- **批量签名交易**：

```kotlin
val base58 = "transaction data"
val address = "wallet address"
val method = SolanaMethod.SignAllTransactions(listOf(SignTransaction(base58, address)))
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is SolanaMethodResponse.SignAllTransactions) {
            Log.d("OKXConnect", "交易列表: ${response.transactions}")
        }
    }
}
```

- **签名并发送交易**：

```kotlin
val base58 = "transaction data"
val address = "wallet address"
val method = SolanaMethod.SignAndSendTransaction(base58, address)
val request = RequestParamsMethod(method = method, chainId = SOLANA_MAINNET)
okxConnect.request(request) { result ->
    result.getOrNull()?.let { response ->
        if (response is SolanaMethodResponse.SignAndSendTransaction) {
            Log.d("OKXConnect", "交易数据: ${response.data}, From: ${response.from}")
        }
    }
}
```

#### 7.3.2 Solana 网络常量

| 网络             | 链 ID                                    | 常量                              |
|------------------|------------------------------------------|-----------------------------------|
| Solana Mainnet   | solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp  | Solana.CHAIN_ID.SOLANA_MAINNET    |
| Soon Mainnet     | soon:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp    | Solana.CHAIN_ID.SOON_MAINNET      |
| Soon Testnet     | soon:4uhcVJyU9pJkvQyS88uRDiswHXSCkY3z    | Solana.CHAIN_ID.SOON_TESTNET      |
| Eclipse Mainnet  | eclipse:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp | Solana.CHAIN_ID.ECLIPSE_MAINNET   |

## 8. 版本历史

### 8.1 版本 1.0.0
- 发布日期：2024-06-01
- 功能：
  - 钱包连接支持。
  - EVM 和 Solana 网络交互。
  - 优化连接稳定性和状态管理。
