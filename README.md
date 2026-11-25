# 渡川 · DuChuan

> 粘贴链接，直接高速下载 —— 一个 Android 网盘 / 视频链接解析与下载应用。

[![Platform](https://img.shields.io/badge/platform-Android-3DDC84?style=flat-square&logo=android&logoColor=white)](#构建)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.1-7F52FF?style=flat-square&logo=kotlin&logoColor=white)](https://kotlinlang.org/)
[![Compose](https://img.shields.io/badge/Jetpack%20Compose-Material%203-4285F4?style=flat-square&logo=jetpackcompose&logoColor=white)](https://developer.android.com/jetpack/compose)
[![License](https://img.shields.io/badge/license-AGPL--3.0-blue?style=flat-square)](./LICENSE)
[![Version](https://img.shields.io/badge/version-0.1-informational?style=flat-square)](../../releases)

**当前版本：0.1**（versionCode 3）· minSdk 24（Android 7.0）· targetSdk 34

## 关于本项目

渡川（Duchuan）在上游 [YunX（云析）](https://github.com/CYQawa/YunX) 的**协议解析、网盘功能和 Android 应用结构**基础上进行了二次开发：

- 沿用上游的网盘 / 视频平台协议解析与分享链接处理能力
- 沿用上游的 Android 应用结构（Kotlin + Jetpack Compose + Room + OkHttp）
- **下载能力迁移到独立的 TurboDL SDK**：上层只依赖 TurboDL 门面，不再直接接触具体下载实现

上游以 GNU AGPL-3.0 开源，本项目作为衍生作品同样以 **AGPL-3.0** 发布。

## 下载体系

### 普通直链下载

TurboDL 用于应用内的普通 HTTP/HTTPS 直链下载（YunGet 等直链下载场景）。下载任务会交由 TurboDL 处理，包括：

- 分片
- 并发调度
- 断点续传
- 重试
- 进度统计
- 下载完成
- 失败回退

### HLS 下载

HLS 下载通过 TurboDL 插件提供。基本流程：

```
M3U8 → 解析 Playlist → 获取 Segment → 提交 TurboDL → 并发下载 Segment → 完成后生成下载结果
```

HLS 下载功能主要面向完整资源下载，而不是实时播放。

### 引擎选择

- **默认引擎：内置分片下载器**（TurboDL 的后端实现）——按平台分片并发、断点续传，保存走系统 SAF / MediaStore，Android 10+ 不需要存储权限；普通直链、网盘下载、应用内更新默认都走它
- **Gopeed 引擎（可选 / 实验性）**：仅建议用于**磁力 / BT** 这类任务；**不随 APK 分发**，需在应用内导入内核后手动切换启用
- **aria2（可选内核 APK）**：实验性，仅支持 `arm64-v8a`

> 提示：设置 →「下载引擎」里可切换。GitHub Release 这类会 302 到 `objects.githubusercontent.com` 的直链，建议保持内置分片下载器。

## 技术栈

| 分类 | 选型 |
| --- | --- |
| 语言 | Kotlin |
| UI | Jetpack Compose + Material 3 |
| 持久化 | Room（凭证与下载任务持久化，凭证字段加密存储） |
| 网络 | OkHttp（网络请求与 HTTP 下载） |
| 下载引擎 | TurboDL（通用下载引擎 SDK，含 HLS 插件）；默认后端为内置分片下载器 |
| 可选内核 | Gopeed（磁力 / BT，需应用内导入）、aria2（实验性，仅 arm64-v8a） |
| 注解处理 | KSP |

## 功能

- **网盘分享解析**：夸克 / UC / 迅雷 / 百度 / 139 / 123 / 115 / 光鸭 / 蓝奏云 / 蓝奏云优享版，自动匹配提取码
- **视频链接解析**：抖音 / 快手 / 微博 / AcFun 免登录直出直链；小红书登录后解析；哔哩哔哩登录后画质更高并经应用内合轨；Pixiv 支持 Cookie 解析
- **TurboDL 下载**：分片并发 + 断点续传，任务持久化，支持按平台设置并发数（默认 32，上限 512）
- **在线预览**：图片 / 视频 / 音频 / 常见文本，逐请求携带 Referer、Cookie，直链不再 403
- **图集两列网格**、批量下载、下载自动归组
- **多平台登录**：WebView Cookie、账号密码、短信验证码、JWT
- **认证备份**：用户口令派生密钥 + AES-GCM 加密导出

## 安装

到 Releases 下载最新 APK。minSdk 24（Android 7.0），targetSdk 34。

## 应用内更新

应用内「设置 → 检查更新」读取本仓库的 GitHub Release：

1. 发布时把 APK 上传到本仓库 Release（asset 名以 `.apk` 结尾，tag 形如 `v0.1` / `v0.1.1`）
2. 应用拉取 `https://api.github.com/repos/XMengXing/-workspace-dist-DuChuan/releases/latest`，比较版本号后弹出更新说明
3. 点「下载」经 TurboDL 把更新包下载到 `Download/`，完成后打开安装

更新源可在「设置 → 网络与更新 → 更新源仓库」改成任意 `owner/repo`，留空则不做更新检查。

发布 Release 示例（本机已登录 `gh`）：

```bash
gh release create v0.1 \
  --repo XMengXing/-workspace-dist-DuChuan \
  --title "渡川 v0.1" \
  --notes "TurboDL 普通直链（分片/并发/续传/重试/失败回退）；TurboDL HLS 插件；默认引擎为内置分片下载器；修复安装权限引导" \
  DuChuan_0.1_debug.apk
```

### 安装权限说明

Android 8+ 首次在应用内安装 APK 时，系统要求授予「安装未知应用」权限。渡川会先弹出说明对话框（而不是直接跳到系统页）：

1. 点「去开启」→ 进入系统的「安装未知应用」页，打开开关
2. 返回渡川，应用检测到权限已授予后**自动继续安装**，无需再次点击

## 构建

```bash
# 需要 JDK 17+、Android SDK（compileSdk 36、build-tools 35）
export ANDROID_HOME=/path/to/android-sdk
./gradlew assembleDebug      # 产物：app/build/outputs/apk/debug/app-debug.apk
./gradlew assembleRelease    # 需自备签名
```

## 免责声明

本应用仅供个人学习与技术交流使用，请勿用于任何商业用途。下载内容版权归原作者所有，请在下载后 24 小时内删除。使用本应用产生的任何后果由使用者自行承担。

## 开源协议

GNU AGPL-3.0。上游版权归 CYQawa 及 YunX 贡献者所有；本仓库的二次开发部分同样以 AGPL-3.0 开放。
