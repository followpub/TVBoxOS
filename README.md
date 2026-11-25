# TVBoxOS

基于 Java 开发的原生 Android 影视聚合播放器，面向电视与机顶盒的横屏、遥控器操作场景。通过 JSON 配置接入内容接口、直播列表和 Spider 插件，使用 ExoPlayer 或 IJK 播放媒体。

- 源码仓库：[q215613905/TVBoxOS](https://github.com/q215613905/TVBoxOS)
- 许可证：[GNU AGPL v3](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/LICENSE)
- 文档核验基线：[`89ec1b7`](https://github.com/q215613905/TVBoxOS/commit/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0)，2026-11-11。

> 本文根据上述提交的源码整理。构建命令和功能说明未经本次编译、设备安装或媒体源实测；其他提交及第三方 APK 可能存在差异。

## 功能

- JSON 配置加载、本地缓存及下载失败后的缓存回退。
- 内容分类、筛选、搜索、详情、线路及剧集选择。
- XML、JSON 和远程 HTTP 内容接口。
- Java/Dex、JavaScript Spider；Python 构建额外支持 Python Spider。
- ExoPlayer、IJK 播放及部分外部播放器调用。
- WebView 嗅探、HTTP JSON 解析及插件扩展解析。
- 直播、字幕、弹幕、收藏和播放进度等功能入口。
- 应用内 HTTP 代理、远程控制和文件传输相关功能。

功能是否可用取决于构建版本、设备、配置及外部接口。项目提供播放与接入能力，不保证默认配置含有可用影视源，也不保证第三方源持续有效。

## 获取安装包

第三方构建仓库 [j4Uq/TVBoxOSC](https://github.com/j4Uq/TVBoxOSC/releases) 提供基于本项目的 APK。它不是本源码仓库的官方发布承诺，下载时请核对发布说明中的源码提交、CPU 架构和插件能力。

需要维护个人版本时，建议从源码构建，使用自己的应用标识、签名和发布记录。

## 工程结构

```text
TVBoxOS/
├── app/                         Android 主应用
│   ├── src/main/
│   │   ├── java/com/github/
│   │   │   ├── catvod/           Spider 接口、加载器及 JS 桥接
│   │   │   └── tvbox/osc/
│   │   │       ├── api/          配置及接口管理
│   │   │       ├── bean/         数据模型
│   │   │       ├── player/       播放器适配
│   │   │       ├── server/       应用内 HTTP 服务
│   │   │       ├── ui/           Activity、Fragment 及控件
│   │   │       ├── util/         工具及播放器配置
│   │   │       └── viewmodel/    内容请求与播放信息处理
│   │   └── res/                  XML 布局、图片及样式
│   ├── src/normal/java/         Java 构建的 Python 空实现
│   └── src/python/java/         Python 构建的 Java 桥接
├── player/                      播放器封装及原生库
├── quickjs/                     QuickJS Android/Java 包装依赖
├── pyramid/                     Chaquopy 与 Python 插件运行支持
├── .github/workflows/           自动构建工作流
├── build.gradle
└── settings.gradle
```

## 构建环境

以下版本来自仓库配置，应优先复现原环境，再单独升级依赖。

| 项目 | 配置值 |
| --- | --- |
| Android Gradle Plugin | 7.2.2 |
| Gradle Wrapper | 7.5 |
| compileSdk | 33 |
| targetSdk | 28 |
| Java 源码/目标级别 | Java 8 |
| Chaquopy | 12.0.1 |
| ExoPlayer | 2.19.1 |
| QuickJS 包装依赖 | 2.1.0 |
| Python 模块 Build Tools | 30.0.3 |

建议从 JDK 11 开始复现这一代构建环境；Java 8 源码级别不等同于必须用 JDK 8 运行 Gradle。Android Studio 的 Gradle JDK、SDK 路径及依赖仓库访问需要按本机环境设置。

### 构建变体

| Flavor | CPU ABI | minSdk | Python |
| --- | --- | --- | --- |
| `java` | armeabi-v7a、arm64-v8a | 19 | 不支持 |
| `java32` | armeabi-v7a | 19 | 不支持 |
| `java64` | arm64-v8a | 21 | 不支持 |
| `python` | armeabi-v7a、arm64-v8a | 19 | 支持 |
| `python32` | armeabi-v7a | 19 | 支持 |
| `python64` | arm64-v8a | 21 | 支持 |

以上为 Gradle 声明的 ABI 和最低 API，实际运行兼容性仍需设备测试。Java 系列仍包含 JavaScript 运行支持；名称中的 Java/Python 主要用于区分是否集成 Python。

### 本地构建

1. Fork 或克隆仓库，在 Android Studio 中打开工程根目录。
2. 配置兼容的 Gradle JDK，安装 Android SDK Platform 33，以及构建提示需要的 SDK 工具。
3. 等待 Gradle 同步；首轮选择 `javaDebug`。
4. 构建并安装到测试设备。

Windows PowerShell：

```powershell
.\gradlew.bat :app:assembleJavaDebug
```

Linux/macOS 构建主机：

```bash
chmod +x gradlew
./gradlew :app:assembleJavaDebug
```

输出通常位于 `app/build/outputs/apk/java/debug/`，项目自定义名称为 `TVBox_debug-java.apk`。

### Python 构建

`pyramid/build.gradle` 当前包含作者机器上的路径：

```groovy
buildPython("D:/Programs/Python/Python38/python.exe")
```

构建前需将其改为本机与 Chaquopy 配置兼容的 Python 可执行文件，并安装所需 Build Tools。不要假定任意最新 Python 版本都可兼容这一旧版工具链。

```powershell
.\gradlew.bat :app:assemblePython64Debug
```

Python 模块声明了 `lxml`、`ujson`、`pyquery`、`requests`、`jsonpath`、`cachetools`、`pycryptodome` 和 `beautifulsoup4` 等依赖；依赖下载与平台兼容问题需要单独排查。

### 发布构建

```powershell
.\gradlew.bat :app:assembleJavaRelease
```

执行 Release 任务不代表已经得到可分发的已签名 APK。自行配置发布签名并妥善备份密钥；不要将私钥或密码提交到仓库。不同签名的 APK 通常不能直接覆盖安装。

## 配置内容源

通过应用设置中的配置入口填入可访问的 JSON 地址。配置中可以分别定义站点、Spider、解析器和直播内容。

下面是结构示例，其中地址与插件名称均为占位内容，不能直接提供播放服务：

```json
{
  "spider": "https://example.com/plugins/spider.jar",
  "sites": [
    {
      "key": "demo_json",
      "name": "示例 JSON 接口",
      "type": 1,
      "api": "https://example.com/api.php/provide/vod/",
      "searchable": 1,
      "quickSearch": 1,
      "filterable": 1,
      "playerType": 2
    },
    {
      "key": "demo_spider",
      "name": "示例 Spider",
      "type": 3,
      "api": "csp_Demo",
      "ext": {}
    }
  ],
  "parses": [],
  "lives": [],
  "flags": []
}
```

### 站点字段

| 字段 | 含义 |
| --- | --- |
| `key` | 站点标识；建议在配置内保持唯一 |
| `name` | 显示名称 |
| `type` | `0` XML、`1` JSON、`3` 本地 Spider、`4` 远程 HTTP |
| `api` | 接口地址、JS/Python 地址或 Java Spider 名称 |
| `ext` | 传给插件或远程接口的扩展内容；对象和数组会序列化为字符串 |
| `jar` | 可选的站点独立 JAR |
| `searchable` / `quickSearch` | 搜索及快速搜索开关 |
| `filterable` | 筛选开关 |
| `playUrl` | 播放解析前缀或相关配置 |
| `playerType` | `1` IJK、`2` Exo；`-1` 使用全局设置 |

`sites` 数组中的项缺少 `key`、`type` 或 `api` 时会被跳过。顶层 `spider` 指定默认主 JAR，不能把它当成站点列表。

> 播放器编号以实现为准：该提交的内部播放器选择中，`0` 等其他值会回退到 Exo，不能将 `0` 理解为已实现的独立系统播放器。

### 加载与缓存

`ApiConfig.loadConfig()` 读取配置地址，优先按调用参数使用缓存，或发起网络请求；下载失败时尝试回退到已有缓存。配置内容经过解码/解密和相对路径处理后，由 `parseJson()` 建立站点与解析器集合。

配置缓存、主 JAR 缓存、站点 JAR 缓存和 JS 模块缓存是不同机制。更新 JSON 后若插件没有更新，需要分别检查缓存及加载器状态。MD5 缓存判断不等同于可信发布者签名验证。

## Spider 插件

统一 Java 接口位于 `app/src/main/java/com/github/catvod/crawler/Spider.java`，包含首页、分类、搜索、详情、播放信息、代理和资源释放等方法。

对于 `type=3`，`ApiConfig.getCSP()` 根据 `api` 选择加载器：

| 条件 | 运行方式 |
| --- | --- |
| 以 `.js` 结尾或包含 `.js?` | `JsLoader` → QuickJS |
| 否则包含 `.py` | `pyLoader` → Python 构建运行时 |
| 其他 | `JarLoader` → Android `DexClassLoader` |

### Java

例如 `csp_Demo` 会映射到 `com.github.catvod.spider.Demo`。加载器创建实例后设置站点 key，调用 `initApi()` 和 `init(Context, ext)`，并缓存实例。

插件运行在应用进程内，可依赖 Android API；扩展名为 JAR 不代表它可以在普通 Linux JVM 上直接运行。只加载可信配置及插件。

### JavaScript

`JsSpider` 将 Java 方法映射为 JS 的 `home`、`homeVod`、`category`、`detail`、`search` 和 `play`，并提供模块加载及宿主桥接。它不是完整的 Node.js 或浏览器环境，脚本需要匹配项目提供的 API。

### Python

Python 构建通过 Chaquopy 启动运行时，下载并导入 Python 模块中的 `Spider` 类，再通过 Java/Python 桥接执行方法。Java 构建中的 Python 加载器为空实现。

## 播放链路

```text
搜索 / 详情
  → SourceViewModel
  → HTTP 接口或 Spider
  → 线路与剧集模型
  → PlayFragment.play()
  → SourceViewModel.getPlay()
  → playerContent() / 远程播放接口
  → 播放结果规范化
  → 直接播放，或解析 / WebView 嗅探
  → PlayerHelper
  → VideoView
  → ExoPlayer / IJK
```

`playerContent()` 返回值可能包含 `url`、`parse`、`jx`、`playUrl`、请求头和字幕等信息。`url` 不保证是最终直链，也可能是多画质数组或需要代理的地址。

`parses[].type` 与 `sites[].type` 是不同枚举：解析器的 `0` 为 WebView 嗅探，`1` 为 HTTP JSON，`2` 为 JAR JSON 扩展，`3` 为聚合解析，`4` 为超级解析分支。

本地代理由 `RemoteServer` 提供，默认端口字段为 `9978`。其 `/proxy` 请求会分发给相应插件，再转换为 HTTP 响应。不要把返回的设备本地地址直接当成其他客户端可访问的地址。

## 维护与排错

| 问题 | 优先检查 |
| --- | --- |
| Gradle 同步失败 | Gradle JDK、SDK、旧依赖仓库及网络连接 |
| Python 构建失败 | `buildPython` 路径、Chaquopy 兼容性、Python 依赖 |
| 配置加载失败 | HTTP 响应、JSON 结构、地址与缓存 |
| Python 源为空 | 当前是否安装 Python 构建 |
| Java Spider 加载失败 | JAR 地址、DEX/Android 兼容性、类名及日志 |
| 有详情但不能播放 | `playerContent()` 结果、解析器、请求头、代理和媒体格式 |
| 别的设备无法访问播放 URL | 是否返回了 `127.0.0.1` 或设备内部代理地址 |

建议每次只修改一个明确问题，分别验证启动、配置加载、搜索、详情、播放、切集、进度恢复和退出重进。问题报告应包含提交号、构建变体、设备/Android 版本、复现步骤及去除敏感信息的日志。

### 个人分支的 CI

核验提交中的 `.github/workflows/test.yml` 会再次克隆 `q215613905/TVBoxOS`，而不是直接编译当前 Fork 的工作区。维护个人分支时，应先调整源码检出逻辑，确保构建的是自己的提交。

该工作流还包含版本号、仓库地址及签名注入操作。直接本地构建与工作流构建的环境并不相同；启用个人 CI 前应审阅这些步骤，并改用自己的发布签名。

## 平台范围

当前工程是 Android 应用，不是现成的 Linux 服务端或 iOS 客户端。

- `type=4` 已提供远程 HTTP 内容接口入口，可作为独立服务端适配的参考。
- Linux 服务端需要重建插件运行环境、媒体代理和会话隔离；Android DEX 插件不能直接按普通 JVM 插件运行。
- iOS 需要重新实现界面与播放适配，可通过服务端复用内容协议；这不意味着兼容所有 Android 插件和媒体格式。

这些是基于源码边界的开发方向，尚无本文提供的跨平台移植实现或运行验证。

## 关键源码

以下链接固定到本文核验提交。

| 模块 | 文件 |
| --- | --- |
| 构建变体 | [app/build.gradle](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/build.gradle) |
| 配置加载 | [ApiConfig.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/tvbox/osc/api/ApiConfig.java) |
| Spider 接口 | [Spider.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/catvod/crawler/Spider.java) |
| Java 插件 | [JarLoader.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/catvod/crawler/JarLoader.java) |
| JavaScript | [JsSpider.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/catvod/crawler/js/JsSpider.java) |
| Python 桥接 | [PythonLoader.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/python/java/com/undcover/freedom/pyramid/PythonLoader.java) |
| Python 模块 | [app.py](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/pyramid/src/python/app.py) |
| 内容与播放请求 | [SourceViewModel.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/tvbox/osc/viewmodel/SourceViewModel.java) |
| 播放页面 | [PlayFragment.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/tvbox/osc/ui/fragment/PlayFragment.java) |
| 播放器选择 | [PlayerHelper.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/tvbox/osc/util/PlayerHelper.java) |
| 应用内 HTTP 服务 | [RemoteServer.java](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/app/src/main/java/com/github/tvbox/osc/server/RemoteServer.java) |
| CI 工作流 | [test.yml](https://github.com/q215613905/TVBoxOS/blob/89ec1b7504426b3ff5e9d0996c3bbf50c9d2eff0/.github/workflows/test.yml) |

## 许可证与致谢

本仓库根目录附有 GNU AGPL v3 许可证。修改、分发或部署衍生实现前，请阅读完整许可证；第三方插件、媒体内容、原生库和依赖的许可应分别核对。

感谢原项目维护者以及 TVBox、CatVod、ExoPlayer、IJK、QuickJS、Chaquopy 等相关项目的贡献者。请保留原有版权和许可证声明，使用具有相应授权的内容源。
