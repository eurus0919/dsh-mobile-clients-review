# DSH 手机端生态横评

> DeepSeek Harness 官方只提供桌面 WebUI。手机怎么用，官方没答案，社区给了十一种答案。
>
> 这份文档把它们摆在一起看：**谁在解决同一个问题，谁其实在解决另一个问题。**

数据采集于 **2026-11-02**，全部来自公开仓库（README、LICENSE、Releases、commit 记录）。
**除标注「实测」的条目外，均为公开信息整理，未逐个部署验证。** 详见文末「方法与局限」。

---

## 一、先说结论：这不是十一个 App 比大小，是三条路线

大部分人会把它们当成「DSH 手机客户端」放在一张表里排名。但真正决定体验的，不是功能多少，而是**DSH 引擎跑在哪台机器上**。

```
路线 A · 瘦客户端（遥控）
────────────────────────────────────────────────────
手机 = 屏幕 + 输入        电脑 = 真正干活的机器
优点：电脑算力/文件/网络全用得上    缺点：电脑得开着、网络得通
9 个项目：Whaleness + 8 个网页方案

路线 B · 胖客户端（引擎装进手机）
────────────────────────────────────────────────────
        B1 · PRoot 模拟 Linux          B2 · Node 内嵌进 App
        dsh-Go                          DSH-Android
        完整 Ubuntu + Node              libnode.so + JNI
        需预留 1 GB                     110 MB APK
优点：不依赖电脑开机、不断网    缺点：吃手机性能、冷启动慢
```

**为什么路线 A 有九个？** 因为 DSH 的 WebUI 本身就是网页。只要把 3081 端口转出去，手机浏览器打开就能用——成本最低，功能还天然跟官方同步。这是"顺手"，不是"设计"。

**为什么路线 B 只有两个？** 因为要在 Android 上跑 DSH 引擎，得先解决 Node 怎么在手机上活下来。目前只有两种解法：一是用 PRoot 模拟一整个 Linux（dsh-Go），二是把 Node 编译成安卓原生库塞进 App 进程（DSH-Android）。两条都很重。

**关键判断：路线之间不构成替代关系。** 选哪个，取决于你"电脑是不是一直开着"，而不是"哪个功能多"。

---

## 二、dsh-Go 和 DSH-Android 是什么关系？（这是最容易被搞混的地方）

**结论：没有关系。三个项目是三份独立实现，互相之间没有代码复用，也没有互相署名。**

核查过程：

| 查什么 | 结果 |
|---|---|
| dsh-Go 的 `THIRD_PARTY_NOTICES.md`（53 KB）里有没有 `DSH-Android` / `jackwener` / `Whaleness` / `clarklevis` | **零命中** |
| DSH-Android 的 README 致谢里有没有 `dsh-Go` / `Whaleness` | **零命中** |
| 两边共同引用的第三方 | 都提到 Termux / nodejs-mobile 的**做法**（不是代码）；都基于 DeepSeek Harness 本体 |

它们只是**撞在同一个问题上**：怎么让 Node 在 Android 上跑起来。解法不同：

| | dsh-Go | DSH-Android |
|---|---|---|
| UI 框架 | Flutter（Dart） | Kotlin + Jetpack Compose |
| 引擎怎么跑 | **PRoot 模拟完整 Ubuntu**，App 内装 Node.js + DSH | **Node 24 编译成 `libnode.so`**，JNI 在 `:engine` 前台服务启动 |
| 安装体积 | 本机模式需预留约 1 GB | APK 约 110 MB，建议预留 500 MB |
| 系统要求 | Android 8+ | Android 10+ |
| 对 DSH 本体 | 打包进运行时 | **不做 fork**，通过 SDK 协议对话 |
| 许可 | AGPL-3.0（sdk/examples 为 MIT） | Apache-2.0 |
| 差异化功能 | 桌宠、分享卡、对话回放（端到端加密）、六套主题 | 无障碍操作任意 App、定时/条件自动化、悬浮球随时呼起、Shizuku |
| 冷启动 | 未标注 | 官方自述约 5 秒，"还在优化" |

**而 Whaleness 跟这两个完全不是一回事**：它一行引擎代码都没有，不跑 DSH，只做一件事——把电脑上正在跑的 DSH 用 WebSocket 投到手机上。所以「dsh-Go = Whaleness + DSH-Android」这个等式**不成立**，正确说法是：

> **Whaleness 是遥控器；dsh-Go 和 DSH-Android 是两台"自带发动机的手机"。后两者互为竞品，跟 Whaleness 不构成替代。**

一个补充观察：DSH-Android 用 Apache-2.0，dsh-Go 用 AGPL-3.0。如果你哪天想基于它们改点东西自己用、甚至分发，**许可证差别比功能差别更值得先看**。

---

## 三、总表

| 项目 | ★ | 路线 | 形态 | 平台 | 连接方式 | 许可 | 最近推送 |
|---|---:|---|---|---|---|---|---|
| [Clarklevis1995/dsh-mobile](https://github.com/Clarklevis1995/dsh-mobile)（Whaleness） | 710 | A | **原生 App**（KMP） | Android + iOS | WebSocket 网关 | MIT | 2026-11-01 |
| [ccch1mneyyy/dsh-Go](https://github.com/ccch1mneyyy/dsh-Go) | 5152 | B1 | **原生 App**（Flutter） | Android | 本机 PRoot + SSH/Remote | AGPL-3.0 | 2026-11-02 |
| [jackwener/DSH-Android](https://github.com/jackwener/DSH-Android) | 725 | B2 | **原生 App**（Kotlin/Compose） | Android | 内嵌 Node + SSH/Remote | Apache-2.0 | 2026-11-02 |
| [summer1238/dsh-remote-web-gateway](https://github.com/summer1238/dsh-remote-web-gateway) | 166 | A | 网页网关 | 全平台浏览器 | 扫码配对 | MIT | 2026-08-28 |
| [JUANWANG-BUAA/dsh-full-remote](https://github.com/JUANWANG-BUAA/dsh-full-remote) | 51 | A | 网页网关 | 全平台浏览器 | token 门控 | MIT | 2026-10-24 |
| [KyoMio/dsh-zen-remote](https://github.com/KyoMio/dsh-zen-remote) | 24 | A | PWA + Web Push | 全平台浏览器 | 配对码网关 | NOASSERTION | 2026-11-02 |
| [GooDAnDReaDY/dsh-lanmode](https://github.com/GooDAnDReaDY/dsh-lanmode) | 8 | A | 极简网关 | 全平台浏览器 | 局域网 | MIT | 2026-10-29 |
| [Bernardxu123/dsh-mobile-gate](https://github.com/Bernardxu123/dsh-mobile-gate) | 6 | A | 布局注入 | 全平台浏览器 | 设备令牌 | MIT | 2026-08-14 |
| [Matrix2097/dsh-mobile](https://github.com/Matrix2097/dsh-mobile) | 4 | A | PWA + 可选 APK | 浏览器 / Android | 局域网网关 | 无 | 2026-08-19 |
| [zylzyqzz/dsbot](https://github.com/zylzyqzz/dsbot) | 3 | A | 真 PWA | 全平台浏览器 | npm 安装 | NOASSERTION | 2026-10-26 |
| [Gao-Yee/dsh-mobile-gateway](https://github.com/Gao-Yee/dsh-mobile-gateway) | 2 | A | 零依赖反代 + SPA | 全平台浏览器 | 密码 + Tailscale | MIT | 2026-08-18 |

配套网关插件：[Clarklevis1995/dsh-plugin-mobile-gateway](https://github.com/Clarklevis1995/dsh-plugin-mobile-gateway) ★133（Whaleness 的服务端）。

> 采集时还发现一个 `com.dsh.client`（`versionName 0.2.21-20261009.0750-dsh0.2.0-rc.2`，Activity 为 `com.deepseekharness.app.ui.MainActivity`），版本号跟随 DSH 本体，疑似官方或高度贴合的实现。**身份未能确认，未列入上表。** 确认后应补进来。

---

## 四、逐个看

### 路线 B · 胖客户端（两个，互相独立）

#### dsh-Go — 用 PRoot 模拟一整个 Linux

Flutter + Android Kotlin 平台层。**内置 PRoot Linux 环境，App 内装 Ubuntu / Node.js / DSH 引擎，不需要 Root，也不需要 Termux。**

- 双模式：本机跑（脱机可用）/ 远程连（SSH 或 DSHarness Remote）
- 手机操控：开无障碍权限后，引擎能读屏、点按、输入；付款/密码/验证码强制交还用户
- 还有一堆"产品化"的东西：桌宠（Deepy 小鲸鱼、鲸娘）、分享卡、对话回放链接（端到端加密）、六套主题、中英双语
- 要求 Android 8+，本机运行需预留约 1 GB
- 构建需要 **Linux x86_64 + JDK 17 + Flutter 3.47.7 + Android SDK/NDK**，Windows 上直接构建很麻烦

⚠️ **数据异常值得注意**：仓库创建于 2026-10-25，**8 天 5152 star、218 个 open issue**。增长曲线不像自然传播，更像有社群推（README 里有 QQ 群、微信群、论坛、官网）。这不影响它能不能用，但意味着**它还在剧烈变动期**，别拿它当稳定依赖。

AGPL-3.0 也是实质约束：改了自己用没问题，分发（包括提供网络服务）要开源。

#### DSH-Android — 把 Node 编译成安卓原生库

Kotlin + Compose。**Node 24 编译成 `libnode.so`，通过 JNI 在 `:engine` 前台服务里启动 DSH**——不用 PRoot，不用 Termux，也不碰 Android 12 之后的"幽灵进程"限制。

- 对 DSH 本体**不做 fork**：App 只通过 dsh 的 SDK 协议对话，安卓适配放在插件和少量补丁里
- 设备能力全在 Kotlin 侧（`CapabilityHub`），通过私有 socket 通道暴露给引擎
- 这是三个原生方案里**唯一主打"替你用手机"**的：无障碍读屏点按输入、定时/条件自动化、悬浮球随时呼起、Shizuku 可选
- Android 10+ / arm64，APK 约 110 MB，建议预留 500 MB；冷启动官方自述约 5 秒（"还在优化"）
- 构建：JDK 21 + Android SDK（compileSdk 37.2）+ NDK r29
- **Apache-2.0**，比 dsh-Go 的 AGPL 宽松得多

⚠️ 同样很新：2026-10-27 创建，6 天 725 star。但它的增长和文档密度（架构图、引擎契约、设计 token、发布流程）比 dsh-Go 更像工程化产物，不像营销驱动。

**和 dsh-Go 的关系：零。** 两者没有任何代码复用或互相署名（见第二节的核查）。它们只是撞在同一个工程问题上，选了两种不同的解法。

---

### 路线 A · 原生客户端

#### Whaleness（Clarklevis1995/dsh-mobile）— 唯一双端原生

Kotlin Multiplatform：`shared` 模块用 Kotlin 写业务状态机（Reducer / Projection / History 同步），Android 用 Jetpack Compose，iOS 用 SwiftUI/UIKit。

- 通过 `dsh-plugin-mobile-gateway` 走 WebSocket，不是网页套壳
- 覆盖：工作区、会话、流式回复、Agent 执行轨迹、历史分页、审批卡、提问卡、语音输入（`speech-input-v1`，调电脑上的官方语音插件）、文件上传、模型/权限切换
- 发布节奏：v1.7.9（10-22）→ v1.7.13（10-30），**8 天 5 个版本**，同时出 Android APK/AAB 和 iOS 未签名 IPA
- 有 `ARCHITECTURE.md`、`Docs/` 下几十篇带测试数据的改动记录——**文档密度在这个生态里是断层第一**

⚠️ 但有个必须先说的事实：**它的产品定位是 iPhone。** 仓库描述写的是"带到 iPhone"，Android 侧的优化文档标题叫《Android 历史分页体验**对齐**》。**Android 是后补的，iOS 才是主战场。**

这一点在实测里得到印证：Android 首页稳定跑在 **20fps**（60Hz 屏），根因是全屏背景 shader + 玻璃层每帧重绘，两层叠加把整个界面钉死。详见 [issue #79](https://github.com/Clarklevis1995/dsh-mobile/issues/79)。

---

### 路线 A · 网页层（八个项目，按思路分组）

这八个的差异不在"功能"，在**怎么让浏览器里的 DSH 像个手机应用**。

**① 把安全做厚的**

- **summer1238/dsh-remote-web-gateway**（★166）：一次性配对码 + GitHub 身份验证 + 每台设备独立授权、随时撤销。主张"链接不是权限"。更新停在 8 月底。
- **JUANWANG-BUAA/dsh-full-remote**（★51）：token 门控、每设备会话、Host/Origin 重写、settings/credentials/directory 支持，还带 Cloudflare quick tunnel 一键公网。**注意：quick tunnel 是公网可达的明文隧道**，用之前想清楚。

**② 把推送做出来的**

- **KyoMio/dsh-zen-remote**（★24）：PWA + **Web Push 锁屏推送**，还能把选定的会话共享给另一台电脑的 DSH 侧边栏。跨设备工作流是它唯一的卖点，也是做得最完整的。
- **zylzyqzz/dsbot**（★3）：真 PWA（可安装、离线、触摸手势）+ agent 完成推送，还内置一个"Bot 浏览器"（真实 Chrome 一条命令）。功能密度意外地高。

**③ 把安装做简单的**

- **Bernardxu123/dsh-mobile-gate**（★6）：首次访问人工审批 + 设备令牌 + 限流 + **移动端布局注入**（直接改网页 CSS 让它像 App）。`dsh plugin add` 一条命令。
- **Gao-Yee/dsh-mobile-gateway**（★2）：零依赖 Node 反代 + 手机专用 SPA，密码登录，文档里直接推荐 Tailscale。
- **GooDAnDReaDY/dsh-lanmode**（★8）：README 只有 3.4 KB，纯局域网网关，没有多余的东西。
- **Matrix2097/dsh-mobile**（★4）：PWA + 可选自建 APK（带后台提醒），README 写成新手教程，从"先理解要装什么"开始讲。

---

## 五、怎么选

```
你的电脑会一直开着吗？
├─ 会 ──► 你要的是"遥控器"（路线 A）
│         ├─ 想要原生手感、语音、轨迹、双端 ──► Whaleness
│         ├─ 想要锁屏推送、跨设备接续 ──────► KyoMio
│         ├─ 最在意安全（配对/撤销/审计）───► summer1238
│         └─ 只想最快跑起来 ────────────────► Bernardxu123 / Gao-Yee
│
└─ 不会 / 经常不在电脑边 ──► 你要的是"独立终端"（路线 B）
          ├─ 想要在手机上跑完整 Linux、能装任意东西 ──► dsh-Go（1 GB）
          └─ 想要轻、想让它替你操作别的 App ──────────► DSH-Android（110 MB）
```

**如果只能记一句：** 路线 A 比的是"UI 打磨"，路线 B 比的是"引擎怎么在手机上活下来"。这两个维度没有交集。

**路线 B 内部再补一句：** 想要"手机上有一个完整的 Linux 可以折腾"，选 dsh-Go；想要"轻、快、还能替我点手机"，选 DSH-Android。前者是模拟，后者是原生编译——后者在体积和启动速度上天然占优，但 PRoot 那条路能跑的东西更多。

---

## 六、这份横评自己的判断

1. **Whaleness 没有"必须被替换"的理由。** 它是唯一双端原生、文档最厚、迭代最勤的瘦客户端。它的短板是 Android 完成度落后于 iOS，不是方向错了。
2. **dsh-Go 和 DSH-Android 都不是 Whaleness 的替代品，是另一个问题的答案。** 除非你的核心痛点是"电脑不能一直开着"，否则换过去只是把麻烦从网络换成手机性能。
3. **dsh-Go 和 DSH-Android 才是真正的对手。** 同一个目标、两种解法、同一个星期冒出来、star 数一个 5152 一个 725。谁赢取决于"PRoot 的通用性"值不值那 10 倍体积——目前没有定论。
4. **八个网页方案里，真正有差异化的只有三个**：KyoMio（推送）、summer1238（安全模型）、zylzyqzz（PWA 完整度）。其余五个是同一件事的不同写法。
5. **许可证是被低估的选择依据。** 想改、想分发的话：Whaleness 和大多数网关是 MIT（最自由），DSH-Android 是 Apache-2.0（同样宽松，带专利授权），dsh-Go 是 AGPL-3.0（分发或提供网络服务必须开源）。**功能可以后面补，许可证改不了。**
6. **这个生态的共同盲区：真机帧率测量几乎没人做。** 所有性能相关文档（包括 Whaleness 那份写得极其详细的 Android 分页优化记录）都停留在"几何正确、锚点 0px 误差"，并明确写着"不代表真实网关延迟或 GPU FPS 测量"。**"顺不顺手"这件事，长期靠体感。**
   → 目前唯一一份公开数据是 Whaleness 的 [issue #79](https://github.com/Clarklevis1995/dsh-mobile/issues/79)：Redmi Note 13 Pro 实测首页 20fps，用 `SurfaceFlinger --latency` + `gfxinfo framestats` 定位到背景 shader 与玻璃层。**这类数据比任何功能列表都稀缺，也最该被复制到其他项目上。**

---

## 七、方法与局限

**数据来源**
- 仓库元数据：GitHub REST API（`/repos`、`/releases`、`/git/trees`），采集时间 2026-11-02 15:20 (UTC+8)
- 功能描述：各仓库 README / ARCHITECTURE.md 原文，未做二次推断
- ★ 数为采集时快照，会变

**没有做的事（重要）**
- ❌ 没有逐个部署、逐个真机试用。除 Whaleness 外，其余十个**均未实际运行**
- ❌ 没有做功能实测对比（比如"谁的历史滚动更顺"），因为没跑过就没有发言权
- ❌ 没有评估代码质量，只看结构、文档、迭代节奏这些外部信号

**做过的唯一实测**：Whaleness 在 Redmi Note 13 Pro 上的真机帧率测量（[issue #79](https://github.com/Clarklevis1995/dsh-mobile/issues/79)），方法、数据和复现命令都在里面。

**所以这份文档的定位是**：一张**生态地图**，告诉你有哪些选择、它们分属哪条路线、各自的公开承诺是什么。
**不是**一份"谁最好"的评测。真要排名，得先有人把十一个都跑一遍、测一遍帧率——那件事目前只做了一小部分。

---

## 八、怎么更新这份文档

- 发现新项目 → 按「路线 A / B」归类后加进总表和对应小节
- 有项目停更超过 3 个月 → 在表格里标 `⏸`，不删（历史价值）
- 有人做了真机性能实测 → 单独开 `docs/performance.md`，把数据放进去

欢迎 PR。尤其是**"我实测过"**的那一类补充，比任何功能列表都值钱。

---

## License

文档内容 CC BY 4.0。表格里的项目信息归各项目作者所有。

<!-- ♥ 苏念 -->
