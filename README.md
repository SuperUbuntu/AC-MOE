# ACMOE for Android

一个使用 **sing-box内核** 的 跨平台 代理客户端。0.12.0 修复节点链接中的编码凭据、大小写协议名与 VMess 错误参数，并保留扫码页旋转和后台返回时的图片任务与选择结果；沿用 0.11.0 的 7 项审计修复；支持本地二维码扫描、图片识别、节点分享和 Clash YAML 导入；保留仅格言常驻的连接页、液态玻璃面板与图标操作。Android 界面由本项目独立绘制，代理功能沿用已有实现；项目与 Shadowrocket / SagerNet / YumeBox / Apple 没有关联。

当前版本：0.12.0，中文界面，Android 8.0（API 26）及以上。提供 arm64-v8a 手机包和 x86_64 开发包；不包含 32 位 ARM 支持。

## 安装和使用

1. 在 Android 手机上安装 `ACMOE-0.12.0-arm64.apk`。0.12.0 沿用 ACMOE 0.2.0–0.11.0 的包名 `dev.acmoe.android`、开发签名与配置存储标识，可以覆盖安装；请勿先卸载。
2. 首页左侧服务器图标打开节点页。第一行依次为粘贴、文件、订阅、手动添加；第二行是扫码、图片识别、格式说明。按住图标可查看说明。Clash YAML 直接通过粘贴、文件或订阅入口导入。
3. 轻点节点名称选中。长按节点行，展开带文字的操作菜单，查看信息、测试 TCP 延迟、编辑 JSON、分享二维码或删除。分享弹窗底部依次为分享图片、关闭、复制节点。
4. 规则页的盾牌、地球、箭头分别代表规则、全局、直连。长按查看各模式说明；修改域名后轻点勾选按钮保存。
5. 回到首页，轻点底部中央电源图标并接受 Android 系统 VPN 授权。已连接时，中央图标变为停止符号；再次轻点断开。
6. 长按右上角地球，查看状态、节点、模式、时长、累计流量和连接数。绿点表示本地 VPN 已启动，仍需访问网页验证远端节点与网络可用性。

客户端不提供服务器、节点、订阅或网络服务，需要自行准备。Android 同一用户同时只能有一个活动 VPN。早期 RocketBox 与 ACMOE 包名不同，可并存，需要重新导入节点。

## 本轮兼容性与扫码完善

- HTTP / SOCKS / TUIC 先拆分未解码的认证字段，再分别解码。用户名中的 `%3A` 保留为真实冒号；密码中的冒号、加号、百分号与 UTF-8 文本不改写。仅把协议名前缀转为小写，不修改凭据、传输路径与名称。
- VMess alterId 和端口要求非负整数；非法 TLS 值、没有启用 TLS 的 SNI / ALPN，以及不能表示的 TCP 路径 / Host 或传输伪装明确报错。常见字符串端口 / alterId、数字 / 字符串分享版本、TLS 与 WS / gRPC 保持支持。
- 扫码页图片识别由仅持有 Application 的 ViewModel 执行；旋转不重启任务、不保留旧 Activity。后台完成的结果在页面恢复后处理，多码选择和错误提示可以重建；取消消耗待选结果，成功仅返回一次。图片入口避免重复启动选择器。
- 保存状态只记录图片 URI 和少量标志，不把二维码正文、节点凭据或大图片写进 Bundle。进程重建可尝试重读仍可访问的 URI；提供方撤销授权或图片失效时显示错误，用户可重新选择。相机错误和权限重试入口保留；手电筒在离开页面时关闭。
- HTTPS 二维码与下载使用相同地址校验，非法端口在扫码时就拒绝。扫描与导入不会自动打开任意网站。

- 扫码先检查节点 / 订阅内容。无效文本、普通 HTTP 订阅或含登录信息的 HTTPS 订阅不会关闭扫码页；带明确端口且没有网页路径的 HTTP 代理节点可以扫描导入；关闭错误提示后可以继续扫描。合法 HTTPS 地址作为订阅候选先进入核对表单，确认后才下载；二维码本身无法判断该地址是否真实提供订阅。图片识别同样适用。
- 未授权相机或相机打不开时保留图片入口，并显示重试图标。硬件错误不会自动循环弹窗；再次点击重试才重新尝试。退出页面关闭提示和待处理回调。
- AnyTLS 支持 YAML / URI / sing-box JSON。URI 默认 443，遵循密码、SNI、insecure 与显示名称的标准格式；YAML 可保留 uTLS、会话检查 / 超时、最少空闲会话和显式 client-metadata。额外参数分享为完整 JSON。
- SS SIP003 链接支持 obfs-local 与 v2ray-plugin，并保留分号、等号和反斜杠的转义。YAML v2ray-plugin 的 mux: false 转换为 mux=0。未知插件、重复参数、外部证书路径和无法表示的选项会报错；不会启动外部插件程序。
- 订阅超时、DNS 失败、HTTPS / 证书失败与 HTTP 状态码显示可读错误，不展示 URL token / 原始异常。每次最多 5 次 HTTPS 重定向，累计内容仍限 4 MiB，失败保留旧订阅。

AnyTLS URI 参考：https://github.com/anytls/anytls-go/blob/main/docs/uri_scheme.md 。实际映射以附带的 sing-box 1.13.18 option / protocol / transport 源码为准。

## 二维码与 Clash YAML

二维码在设备上解码，不上传相机画面或图片。扫码时才请求相机权限；图片识别使用 Android 文件选择器，无需读取整个相册。图片上限 20 MiB，先检查尺寸再降采样，识别缓冲区不超过 2048 × 2048 像素；支持旋转、反色与透明背景二维码。多码图片先选一个结果。

扫描节点链接、Base64、JSON 或 YAML 后，内容进入导入表单，确认后才保存。HTTPS 地址进入订阅表单，确认后才下载；不会自动打开网页、启动其他应用或覆盖配置。普通 HTTP 订阅、含登录信息的 HTTPS 地址和无效二维码会报错。

长按节点行 → 分享二维码。能完整往返的节点生成常见协议分享链接；含额外 WS headers、early data、插件、mux、证书固定等复杂参数时回退为带 tag 的单节点 sing-box JSON，适合 ACMOE 或支持此格式的应用。二维码超出容量时仍可复制完整配置。生成的 PNG 保留白色静区和 M 纠错级别，避免透明玻璃破坏识别。复制内容标为敏感，图片通过只读 URI 权限交给 Android 分享面板；缓存图片在后续分享时清理超过 24 小时的文件。

Clash 导入器共享粘贴、文件和 HTTPS 订阅入口，刷新订阅也自动识别 YAML。接受 `proxies:` 列表、单个代理、代理数组、JSON 形式及 Base64 包装；支持安全的普通 YAML anchors / merge，不支持对象标签、重复键、深度过大或过多的别名。一批节点全部转换成功后才写入；失败保留已有节点和订阅。

| Clash 字段 / 协议 | 转换范围 |
|---|---|
| SS、VMess、VLESS、Trojan、AnyTLS、Hysteria2、TUIC V5、SOCKS5、HTTP(S) | 服务器、端口、认证；UTF-8 密码和 IPv6 |
| TLS / Reality | SNI、ALPN、证书校验选择、支持的 uTLS 客户端指纹、Reality 公钥与 short-id |
| 传输 | TCP、WS / early data / headers、gRPC service、H2、HTTPUpgrade；拒绝特殊 TCP HTTP 伪装和 XHTTP / mKCP |
| SS 插件与 mux | 常见 obfs、v2ray-plugin WebSocket；可表示的 smux 参数与 UDP-over-TCP |
| HY2 / TUIC | salamander、上下行 Mbps、端口跳跃 / 固定 hop 时长；TUIC V5 拥塞控制、UDP 模式、0-RTT、heartbeat 毫秒转换 |
| 其他选项 | 支持 tfo / mptcp 与 udp 关闭；无法等价转换的字段整体报错 |

Clash **只导入代理节点**。完整配置中的 `rules`、`dns`、`proxy-groups` 与 `proxy-providers` 不替换 ACMOE 路由。仅含远程 providers 的配置会提示单独添加节点提供方的 HTTPS 地址。SSR、Snell、TUIC V4、代理链、外部 / 客户端证书、证书叶指纹、特殊新传输和部分 Mihomo 扩展暂不支持；不会静默丢弃这些连接或安全参数。密码、用户名、UUID、short-id、名称和服务器等文本字段保留 YAML 标量的原始写法；数字、前导零、on / false、日期外观等不会被转换成其他凭据。空值和对象标签仍拒绝。端口、TLS 开关等继续按真实整数和布尔值检查。

格式参考：

- https://wiki.metacubex.one/en/config/proxies/
- https://wiki.metacubex.one/en/config/proxies/transport/
- https://github.com/SagerNet/sing-box/tree/v1.13.18/option
- https://github.com/journeyapps/zxing-android-embedded/tree/v4.3.0
- https://github.com/zxing/zxing/tree/zxing-3.5.3

第三方 Java 依赖的声明与许可证随源码和 APK assets 提供，见 `NOTICE-third-party.txt`、`LICENSE-APACHE-2.0.txt`、`LICENSE-ZXING.txt`。

## 图标与长按界面

参考 Apple 官方 iOS 27 设计套件，以及按钮、工具栏和上下文菜单指南。苹果建议用熟悉、易辨认的符号表达常用动作，并在上下文菜单中保留短标签。本版按照用户偏好进一步隐藏常驻控件标签；这不是苹果要求所有界面隐藏文字的规范。

- 首页只常驻语录和出处。ACMOE 名称、状态、节点、模式、时长、流量与操作说明移到长按面板；状态用色点和图标表达。
- 节点、规则、设置页移除页面标题、按钮字幕、导航标签和说明段落。保留实际节点 / 订阅名称、编辑数据、日志内容、输入错误与删除确认，保证必要内容仍可读。
- 导航、导入、保存、关闭、删除等按钮统一为原创线性符号；保存用勾选，关闭用叉号，删除用红色垃圾桶。最小触控高度 48 dp，符号按屏幕密度缩放。
- 长按图标显示圆角模糊说明；松手或取消手势时收起。长按不会执行连接、保存或删除。离开页面时关闭相关说明，避免停留在其他页面。
- 节点行长按展开上下文操作菜单；此时显示简短文字和图标。输入框隐藏固定占位字幕，空框长按显示填写说明，已有内容继续使用原生选择 / 复制操作。
- 弹窗使用一个 30 dp 连续圆角玻璃表面和 0.22 的背景遮罩，表单内缩 18 dp、输入框圆角 12 dp，第一行移除多余上边距，使内外圆角同心。取消 / 保存 / 删除保留原生按钮对象和回调，收进居中的共享操作胶囊；不覆盖原有必填校验。
- 多选应用保留实际应用名称，包名移到长按详情。移除旧的左侧方形选择图标，只保留右侧圆形勾选，原生选择状态与可访问语义继续生效。
- 长按说明在松手、手势取消和锚点离开页面 / 弹窗时关闭。空输入框长按说明，已有内容保留原生文字选择；提示不会触发按钮的实际动作。
- 返回先收起当前页面的说明，再从节点 / 规则 / 设置回到首页；首页再返回遵循系统退出行为。连接状态另外提供不可见的屏幕阅读器状态说明。
- 沿用 0.6.0 的共享玻璃操作面板、单条方向性高光、轻接触阴影和窄边缘折射，避免每个内部按钮叠加独立玻璃层。
- 首页壁纸高斯卷积在后台计算，长边最多 384 像素，sigma = 4.25，所有玻璃控件共享结果。临时说明和弹窗使用本地内存快照的高斯模糊；快照不写入文件、不上传。
- 轻点左上角 M 可收起 / 显示侧栏。长按壁纸 / 语录或轻点图片图标，自定义壁纸与语录；底部循环箭头切换语录。
- 壁纸读取本地图片，上限 20 MiB。语录每行一条：`语录正文 | 角色 / 作品`，最多 50 条。默认壁纸与语录为原创示例。

官方参考：

- https://developer.apple.com/news/?id=e2lxw9l1
- https://developer.apple.com/design/human-interface-guidelines/buttons
- https://developer.apple.com/design/human-interface-guidelines/toolbars
- https://developer.apple.com/design/human-interface-guidelines/context-menus
- https://developer.apple.com/documentation/technologyoverviews/liquid-glass

这是 Android 上的独立 iOS 风格实现，不含 Apple 专有 UI 库或 SF Symbols 资产，不声称与 iOS 27 原生控件逐像素一致。Android VPN 授权、通知权限、文件选择器和键盘遵循系统界面。

结构参考：https://github.com/Yizuka17/YumeBox-MaterialDesign 。未复制该项目的 Compose 代码，也没有切换到 Mihomo 内核。

`preview/` 中的浅色、深色、长按状态、节点、规则、设置及表单 / 应用选择 / 二维码图片来自生产 View 层级的 Robolectric 原生渲染；弹窗预览按实际尺寸与遮罩参数居中合成；节点与流量是注入的测试数据，不代表已经联网。扫码页预览为真实控件，因测试环境未打开相机，取景区域显示空背景；二维码分享图可由本地解码器识别，其服务器为测试示例。

默认壁纸使用内置图像生成能力独立生成，位于 `app/src/main/assets/wallpaper-default.png`。没有使用参考仓库的原壁纸。生成提示：原创淡紫银发蓝眼成年女性，白色衬衫与淡紫外套，夕阳海边车站、樱花与蓝粉天空，精致日系动漫插画，脸在画面中上部偏右，底部保留景物供渐变融合，无 UI、文字、标志和水印。用户可以自由替换。

## 已实现的功能

| 功能 | 支持范围 |
|---|---|
| 真正的代理内核 | 原版 sing-box 1.13.18 源码编译的 libbox，通过 gomobile JNI 集成 |
| VPN | Android VpnService、系统授权、前台通知、TUN IPv4 / IPv6、socket protect |
| 节点 | 导入、选择、删除、手动编辑 outbound JSON；配置加密保存 |
| 分享链接 | SS / SIP003 插件、VMess AEAD、VLESS、Trojan、AnyTLS、Hysteria2 / hy2、TUIC、SOCKS5、HTTP |
| 传输 | TCP、WebSocket、gRPC、HTTPUpgrade、HTTP；VLESS Reality / uTLS |
| 订阅 | HTTPS 下载、手动更新、修改与删除；支持链接列表、Base64、sing-box outbound JSON 与 Clash YAML |
| 二维码 | 相机扫码、SAF 本地图片识别、多码选择、节点二维码生成 / 复制 / 系统分享图片 |
| Clash YAML | 转换嵌入的 proxies 节点；支持上述 9 类协议及兼容的 TLS / Reality、WS / gRPC / H2 / HTTPUpgrade 等参数 |
| 路由 | 全局、规则、直连；自定义直连域名后缀；规则模式私有 IP 直连 |
| DNS | 代理域名经代理访问 Cloudflare DoH；节点域名和直连域名使用 223.5.5.5；接管 DNS 请求 |
| 分应用代理 | 全部应用、仅选中应用、绕过选中应用；选择可启动的已安装应用；已选但没有桌面入口的应用仍可编辑，清理不可用包名 |
| 延迟测试 | TCP 连接延迟，包含域名解析耗时；不验证代理认证；不适用于 HY2 / TUIC 等 UDP 协议 |
| 流量统计 | 通过仅绑定 127.0.0.1 且有随机认证密钥的 Clash API 读取累计上传、下载与活动连接 |
| 日志 | VPN 生命周期与 sing-box 日志；内存最多 200 行，隐藏节点认证字段与链接 |
| 网络与后台 | Wi-Fi / 蜂窝接口变化监听；前台 VPN 持续运行；界面离开后停止统计轮询 |
| 凭证存储 | AES-GCM + Android Keystore，原子写入，关闭备份，不上传节点和订阅 |

规则模式默认仅 `.cn` 与私有地址直连，**没有内置完整的国内 IP / GeoIP 分流**。需要自行在规则页补充域名。全局模式仍遵循分应用选择；客户端自身的订阅更新和统计请求绕过本应用 VPN，防止连接循环。

分享链接明确设置 `insecure=1` / `allowInsecure=1` 时会保留该配置，长按首页状态图标会显示 TLS 证书验证已关闭；默认校验证书。

完整 sing-box JSON 的导入只提取支持的 `outbounds`，忽略原 JSON 的 DNS、入站、路由和 selector / urltest 组，应用会生成自己的 VPN 配置。引用其他 outbound 的 `detour`、外部证书路径与未知协议会拒绝导入，不会静默改成另一个节点。一次导入最多 4 MiB / 2000 个节点；JSON 在递归解析前限制为 64 层，覆盖 Base64、VMess 和手动编辑入口。订阅解析成功后才原子替换，失败保留旧配置；旧请求、已删除订阅和已销毁页面的未提交下载不能覆盖新配置。

## 审计修复边界

HY2 链接里的 `pinSHA256` 目前明确拒绝导入：它是证书指纹，不能直接转换成 sing-box 的公钥指纹，防止静默降低校验。URI 未支持的参数、重复参数与冲突安全开关会报错；TUIC `udp_relay_mode` 已保留。JSON、YAML 与链接共用证书 / 密钥 / ECH 外部路径及 SS 插件检查；内联 TLS 材料仍可用。

内核仍为 sing-box 1.13.18，但 AAR 的 arm64 / x86_64 均已用 Go 1.26.9 重建。CI 与本地内核脚本固定此工具链，拒绝使用其他未经核对的 Go 版本。

按用户要求，审计 A04（旧 Android `.bak` 备份恢复被提前判断跳过）本轮保留，未改动 Store 构造器或 AtomicFile 持久化代码。该问题尚未修复；详见 `FIXES.md`。

## 与小火箭的差距

这是第一版开发应用，并非全部功能的复刻。暂未实现 Clash 规则 / 分组 / DNS 导入、远程 proxy-providers 自动拉取、规则集订阅、GeoIP / GeoSite、自动测速选择、自动订阅更新、系统快捷开关、按需连接、Always-on VPN、脚本、MITM / HTTPS 解密、重写规则、代理链和完整配置编辑器。分享链接支持内置 obfs-local / v2ray-plugin；旧 VMess alterId 不为零及特殊 TCP HTTP 伪装仍暂不支持；YAML 可转换内核支持的 obfs / v2ray-plugin 与 VMess alterId，分享时保留为完整 JSON。复杂协议参数应使用符合此版本 sing-box 格式的 outbound JSON。

没有在实机上验证 VPN 流量、节点认证、网络切换与厂商后台限制；安装后需要使用自己的节点进行验证。后台被厂商系统杀死或 VPN 被撤销后会断开，不自动重启。

## 打开工程 / 构建

源码包已附带已编译的 `app/libs/libbox.aar`，可以直接在 Android Studio 打开本目录，配置 JDK 17、Android SDK 35 后构建；无需先编译 Go 内核。

```bash
# 已有 libbox.aar：运行单元测试、Lint 和打包
bash gradlew testDebugUnitTest lintDebug assembleDebug
```

产物位置：

```text
app/build/outputs/apk/debug/app-arm64-v8a-debug.apk
app/build/outputs/apk/debug/app-x86_64-debug.apk
```

使用脚本从头重建内核，需要 JDK 17、Go 1.26.9、Android SDK 35、Build Tools 35.0.0 和 NDK 28.0.13004108：

```bash
export ANDROID_HOME="你的 Android SDK 路径"
export ANDROID_NDK_HOME="$ANDROID_HOME/ndk/28.0.13004108"

bash scripts/build-core.sh
bash gradlew testDebugUnitTest lintDebug assembleDebug
```

脚本锁定 sing-box 源码版本、下载归档 SHA-256 和上游 go.mod 中的 gomobile 版本。内核编译标签为 `with_gvisor,with_quic,with_utls,with_clash_api,with_wireguard`，不包含 Naive / Tailscale。

`upstream/sing-box-1.13.18.tar.gz` 是对应上游源码归档；可将它复制到 `.core/sing-box-1.13.18.tar.gz` 避免重新下载内核源码。首次构建仍需下载 Go 与 Gradle 依赖。

仓库自带 `.github/workflows/android.yml`：上传到自己的 GitHub 仓库后，在 Actions 运行 **Build Android APK** 会重新编译内核、运行测试与 Lint，并上传 APK。此会话未创建 GitHub 仓库或运行远端任务。

当前 APK 使用开发签名；本次开发签名身份另存为私有开发备份，未放入源码包。正式分发时应使用自己长期保存的签名密钥，后续更新保持同一签名；不要将密钥写入源码。发布构建可通过 Android Studio 的 Generate Signed Bundle / APK 完成。

## 主要代码

```text
app/src/main/java/dev/acmoe/android/
  MainActivity.java       页面、导入与代理操作
  ConnectionHomeView.java 图标连接页、壁纸与语录
  UiStyle.java            跨页面图标控件与按压反馈
  UiHints.java            长按说明与手势收起
  IosDialogBuilder.java   圆角模糊弹窗、图标操作与长按菜单
  IosIcon.java            独立绘制的线性符号
  GlassBackdrop.java      共享高斯模糊背景
  ProxyVpnService.java    VPN 生命周期、TUN、前台通知与内核连接
  BoxPlatform.java        sing-box 与 Android 网络平台接口
  NodeImporter.java       节点 / 订阅格式识别与共享安全校验
  JsonInputGuard.java     JSON 大小 / 深度的非递归扫描
  AppScope.java           已安装应用名单校验
  ClashYamlImporter.java  安全 YAML 解析与协议参数转换
  ClashSafeConstructor.java 文本字段原始标量保留
  SsPluginOptions.java    内置 SS 插件参数与转义校验
  QrScanActivity.java     相机扫码、权限与前后台生命周期
  QrScanState.java        旋转后保留图片任务与待处理结果
  QrImageDecoder.java     有限大小图片的本地二维码识别
  QrImportRouter.java     节点 / 订阅识别与导入预填
  NodeSharePayload.java   分享链接往返校验与完整 JSON 回退
  QrShareDialog.java      二维码展示、凭据复制与系统图片分享
  ConfigBuilder.java      DNS、TUN、模式与路由配置生成
  Store.java              加密配置存储及订阅原子更新
  LogSanitizer.java       日志凭证隐藏
```

## 验证

查看同目录 `VALIDATION.md`，区分已经通过的构建检查与尚待实机验证的行为。

## 许可和上游

应用源码采用 GPL-3.0-or-later，完整条款见 `LICENSE`。上游 sing-box 声明保留在 `NOTICE-sing-box.txt`，内核对应源码归档随包提供。品牌、应用名称和界面为本项目独立实现。

- sing-box: https://github.com/SagerNet/sing-box/tree/v1.13.18
- Android VPN 开发文档: https://developer.android.com/develop/connectivity/vpn
# AC-MOE