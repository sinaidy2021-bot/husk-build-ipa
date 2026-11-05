# Husk 云端构建包 → 未签名 IPA → 个人证书签名

把这个文件夹里的内容原样推到一个 GitHub 仓库，GitHub 的 macOS 机器会从源码
构建出 `Husk.ipa`。引擎是开源项目 [Husk](https://github.com/Leviidev/Husk)
（GPL-2.0-or-later，底层 QEMU 来自 UTM 分支）：转译层让安卓游戏的 ARM64 代码
直接在 iPhone 芯片上跑，完整系统模式则用 QEMU 启动 LineageOS。
本仓库只含构建流水线，不含引擎源码；构建时按锁定版本现拉官方源码。

锁定版本：`d92d0a112a391833584ac61f21296e566e404cb8`（2026-11-03 主线 HEAD，
构建脚本与工程结构已按此版本核对）

## 操作步骤

### 1. 建仓库并上传
在 GitHub 新建一个仓库（公开或私有均可），把本文件夹内容推上去，保持目录结构：

```
.github/workflows/build-husk-ipa.yml
README.md
```

用网页上传也可以：仓库页 → Add file → Upload files，把 `.github` 文件夹和
`README.md` 一起拖进去（注意 `.github` 是隐藏名，别漏）。

### 2. 触发构建
仓库页 → Actions → 左侧 **Build Husk IPA** → **Run workflow** → 直接 Run
（版本输入框留空就是锁定版本）。
也可以什么都不做：第一次 push 这个 workflow 文件时会自动触发一次。

### 3. 等构建完成，下载 IPA
首次构建要现编译 QEMU、ANGLE、glib、MoltenVK 等全套依赖，**按 1–3 小时预留**
（GitHub 单个 job 上限 360 分钟；超时或失败先看同批上传的 `build-log` 再重跑，
依赖源码有缓存，第二次会快一些）。
构建成功后在该次运行页面底部 Artifacts 下载 **Husk-ipa-unsigned**，
解压得到：
- `Husk.ipa` — 常规侧载版（adhoc 签名，即实际意义上的未签名包，等你重签）
- `Husk.tipa` — TrollStore 版（用不上可忽略）

### 4. 用你的个人证书签名安装
用你惯用的签名方式重签 `Husk.ipa` 装到 iPhone 13 上即可，例如 ESign 类证书
签名工具、或 SideStore / AltStore 安装时自动重签。包名 `com.husk.app`，
系统要求 iOS 15.5 以上。
官方打包脚本的说明：标准侧载重签时，`dynamic-codesigning` 这类个人证书
签不出来的权限会被自动丢弃，这是正常现象，不影响安装。

### 5. 装好之后：JIT 二选一
- **开 JIT（性能完整）**：iOS 26 以上用 App 内置的 StikJIT（设置里按引导走，
  需要 LocalDevVPN + 配对文件，iOS 27 可在手机上直接生成配对文件）；
  或另装 StikDebug 给它开。
- **不开 JIT（用你的证书玩）**：Husk 有一条官方的 “Play without JIT” 路径——
  把安装 Husk 用的那张证书导入 App 后，转译层游戏可以免 JIT 运行。
  你正好有个人证书，这条路对你最省事。完整系统（LineageOS 模拟）仍然需要 JIT。

## 预期效果（先说清，免得落差）
- 转译层支持的游戏/App（Unity、Unreal 4、Godot、cocos2d-x、SDL 等引擎，
  官方兼容列表 150+ 款，含 GTA: San Andreas、Minecraft 系列）：秒开、接近原生速度。
- 完整 Android 系统：QEMU 软件模拟，慢，只适合尝鲜和兜底，别指望日常使用。
- 没有 Google Play 服务：登录、内购类功能用不了。
- iPhone 13 的 4GB 内存是硬约束：大型游戏和完整系统模式都吃紧，闪退先降画质/关后台。

## 出问题先看哪里
- Actions 失败：下载同批的 `build-log` artifact，看最后 50 行的报错；
  多数是上游依赖下载失败（重跑即可）或 Xcode 大版本变动（把 workflow 里
  Husk 版本输入框填一个新 tag 重跑）。
- 装上打不开 / 一开游戏就退：十有八九是 JIT 没开或证书没导入，按第 5 步二选一处理。

## 许可
Husk 及其构建产物为 GPL-2.0-or-later；本构建流水线可自由使用。
产物不能上架 App Store（苹果规则与 GPL 均不允许），仅供个人侧载使用。
