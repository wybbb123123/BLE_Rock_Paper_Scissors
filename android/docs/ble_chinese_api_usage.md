# BLE 中文 API 使用说明

本阶段新增 `:ble_chinese_api` Android library module，并新增 `:sample_app` 作为最小文本收发示例。第一阶段的设计文档已由 Uoniiee 完成，本阶段主要把其中的中文 API 入口落到当前工程里，方便后续把 BLE_BBS 和 BLE_Rock_Paper_Scissors 的通信层逐步抽出来复用。

## 模块接入

业务 App 在 `settings.gradle.kts` 中包含 module 后，增加依赖。

```kotlin
implementation(project(":ble_chinese_api"))
```

当前工程已经包含：

```text
:ble_chinese_api
:sample_app
```

## 权限

业务 App 负责申请运行时权限，library 不直接弹权限框。Android 12 及以上需要：

```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

Android 11 及以下需要：

```xml
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30" />
```

调用 `启动()` 时，如果蓝牙关闭、设备不支持 BLE、缺少权限或 UUID 格式不对，library 会返回 `启动结果.失败`。

## 最小接入

```kotlin
val 通信器 = 蓝牙通信器(
    上下文 = applicationContext,
    配置 = 蓝牙通信配置(
        服务UUID = "f77d0a4b-2b74-4e43-a9de-6cb27a0f7a91",
        写入UUID = "1bb5f2d4-9f73-4f47-a351-9e0d2b3517cf",
        厂商编号 = 0x1234,
        应用标记 = "ble-chinese-api-sample",
    ),
    编解码器 = 文本消息编解码器(),
)
```

启动和停止：

```kotlin
val 结果 = 通信器.启动()
通信器.停止()
通信器.释放()
```

订阅状态和消息：

```kotlin
通信器.连接状态流.collect { 状态 ->
    // 更新界面状态
}

通信器.邻机状态流.collect { 邻机列表 ->
    // 显示附近设备和可写数量
}

通信器.收到消息流.collect { 消息 ->
    // 交给业务层处理
}
```

发送消息：

```kotlin
when (val 结果 = 通信器.发送("你好")) {
    发送结果.已写入 -> Unit
    is 发送结果.失败 -> println(结果.原因)
}
```

`发送结果.已写入` 只表示 Android BLE 写入请求已被当前可写连接接受，不代表对端业务层已经处理完成。业务级确认、重试和历史补发仍由业务 App 决定。

## 自定义业务消息

library 不理解 BBS 帖子或猜拳规则，只关心 `ByteArray` 载荷。业务 App 可以定义自己的消息类型，并实现 `消息编解码器`。

```kotlin
data class 对局消息(
    val 局编号: String,
    val 事件: String,
    val 发送方: String,
)

class 对局消息编解码器 : 消息编解码器<对局消息> {
    override fun 编码(消息: 对局消息): ByteArray {
        TODO("业务 App 可用 JSON 或自定义格式编码")
    }

    override fun 解码(数据: ByteArray): Result<对局消息> = runCatching {
        TODO("业务 App 自行解析")
    }
}
```

此时通信器类型为：

```kotlin
蓝牙通信器<对局消息>
```

## 示例 App 验收

构建示例：

```bash
bash ./gradlew :sample_app:assembleDebug
```

两台 Android 设备安装同一个 `sample_app` APK 后：

1. 两端授予蓝牙相关权限并打开蓝牙。
2. 两端点击启动通信。
3. 至少一端显示可写邻机数量大于 0。
4. A 发送文本，B 能看到对端消息。
5. B 发送文本，A 能看到对端消息。
6. 点击停止通信后不再发送。

## 当前限制

- 当前 library 聚焦最小文本收发，不处理业务历史同步。
- 当前单次载荷上限默认 180 字节，暂未在 library 内做完整分片重组。
- 当前不提供 mesh 策略、后台长期扫描、加密鉴权和业务级 ACK。
- BLE_BBS 与 BLE_Rock_Paper_Scissors 后续可以逐步迁移到该接口，但本阶段先不大规模拆改现有业务代码。
