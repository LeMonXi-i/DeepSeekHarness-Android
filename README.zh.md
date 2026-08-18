# DeepSeekHarness（安卓端）

> **安卓端使用** — [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（`dsh`）的安卓客户端。

一个轻量的安卓外壳 App，把手机变成你自己 dsh 服务器的"远程显示器"：

- **SSH 隧道**（JSch）在手机与服务器之间建立加密连接，
- **全屏 WebView** 渲染 *真实的* dsh 网页界面——100% 功能（Agent 循环、模型切换、插件、权限审批、轨迹、设置），
- 模型 API 密钥**永不离开服务器**。

一句话：**手机当显示器，服务器当主机，SSH 当安全的线。** 只套壳连接，不套壳内容。

[English](README.md) | 中文

## 为什么这样做

| 方案 | 问题 |
| --- | --- |
| 原生重写聊天 UI（v2.x） | 审批、插件、轨迹、模型设置等几十个功能根本做不完 |
| WebView 直连（v3.x） | ✅ 网页端 100% 功能；App 只管连接和外壳 |

## 工作原理

```
┌───────────────┐   SSH 隧道（加密）          ┌──────────────────┐   HTTPS/JSON   ┌────────────────┐
│ 安卓 App      │ ─────────────────────────► │ 服务器：dsh web  │ ─────────────► │ DeepSeek API   │
│ (JSch +       │   手机 127.0.0.1:6080      │ (127.0.0.1:3080) │   （模型/工具…） │ （云端）        │
│  WebView)     │   ──► 服务器 127.0.0.1:3080 │  [cordis + typert]│                 │                │
└───────────────┘                            └──────────────────┘                └────────────────┘
```

1. App（包名 `com.dshtunnel.app`）用 JSch 与服务器建立 SSH 连接，把手机 `127.0.0.1:6080`
   转发到服务器 `127.0.0.1:3080`。
2. 全屏 WebView 打开 `127.0.0.1:6080` —— 看到的就是你真实的 dsh 网页界面。
3. 隧道断开自动重连；App 通过 dsh API（`DshApi`：事件 / 模型 / 预设）实时刷新连接状态。

### 配套常驻机制

- **PC 端**：`login.bat` → 隐藏窗口守护脚本 → 断线 5 秒自动重连 + 开机自启（Run 键）。
- **服务器端**：`systemd` 服务（`dsh-web.service`）→ 开机自启 + 崩溃自动拉起。

### 安全性

- 全程由 SSH 隧道加密传输。
- 模型 API 密钥**只存在服务器**，手机端从头到尾不接触。
- 服务器上配合安装 [dsh-client-ui-android](https://github.com/LeMonXi-i/dsh-client-ui-android)
  插件后，网页界面会自动适配手机布局（抽屉侧边栏、全屏设置、单行聊天工具栏）。

## 下载

从 [Releases](../../releases)（或本仓库 `releases/` 目录）获取最新 APK：

- `releases/DeepSeekHarness-v3.4.1.apk` —— Android 7+（API 24+），约 1.9 MB。

> 仅安卓端使用，不支持 iOS；iOS 用户可直接用浏览器打开网页界面。

## 使用方法

1. 在安卓手机上安装 APK。
2. 填写服务器地址、SSH 端口、用户名、密码/私钥。
3. 点击连接——隧道建立，dsh 界面在 WebView 中加载。
4. 隧道断开自动重连。

## 从源码构建

> 源码将在后续提交中发布（当前版本附带的是编译好的 APK）。App 是标准 Android Studio
> 工程：Kotlin + JSch（SSH）+ WebView（渲染）+ `DshApi`（状态轮询）。

第三方库：[JSch](http://www.jcraft.com/jsch/)（BSD）、JZlib（BSD）、jBCrypt（ISC）——
详见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

## 关联项目

- [dsh-client-ui-android](https://github.com/LeMonXi-i/dsh-client-ui-android) —— dsh 网页界面的安卓/移动端适配插件（配合使用）。

## License

MIT —— 详见 [LICENSE](LICENSE)。
