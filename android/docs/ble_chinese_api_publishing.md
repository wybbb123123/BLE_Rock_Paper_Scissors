# BLE 中文 API 发布说明

当前 `:ble_chinese_api` 先作为本仓库内的 Android library module 使用。后续如果要发布给其他仓库，可以按下面方式演进。

## 本地 module 使用

同一 Gradle 工程内优先使用 project 依赖：

```kotlin
implementation(project(":ble_chinese_api"))
```

这种方式适合 BLE_BBS 和 BLE_Rock_Paper_Scissors 继续迭代，因为调试 API 和业务代码最直接。

## 本地 Maven 发布

如果要在多个本地工程之间复用，可为 `:ble_chinese_api` 增加 `maven-publish` 配置。

建议坐标：

```text
groupId，com.wuxuan
artifactId，ble-chinese-api
version，0.1.0
```

发布到本地 Maven 后，其他工程可以：

```kotlin
repositories {
    mavenLocal()
}

dependencies {
    implementation("com.wuxuan:ble-chinese-api:0.1.0")
}
```

## 版本划分建议

`0.1.x` 保持最小可用能力：

- 中文公开 API 稳定。
- BLE 扫描、广播、连接、写入、接收可跑通。
- 文本消息示例可运行。

`0.2.x` 可增加：

- library 内部分片与重组。
- 业务无关的传输帧编号和去重。
- 可选写入重试。
- 更清晰的错误码。

`1.0.0` 前建议完成：

- BLE_BBS 或 BLE_Rock_Paper_Scissors 至少一个业务 App 迁移到该 library。
- 真机双端收发验证记录。
- API 兼容性说明。
- 发布包 ProGuard consumer rules 检查。

## 发布前检查

建议每次发布前执行：

```bash
bash ./gradlew :ble_chinese_api:assembleDebug :sample_app:assembleDebug
```

如果要连同当前主 App 一起回归：

```bash
bash ./gradlew :app:assembleDebug :sample_app:assembleDebug
```

## 与第一阶段设计的关系

第一阶段已经完成 API 分层和命名设计，并已单独结算。本阶段实现的是第二阶段，范围是把中文 API 作为 Android library module 落到当前仓库，并提供一个最小示例和发布说明。
