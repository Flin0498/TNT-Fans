# StarTrack·星途

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)](#)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)](#)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](#)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore%20%7C%20FCM-FFCA28?style=flat-square&logo=firebase&logoColor=black)](#)
[![Git LFS](https://img.shields.io/badge/Git-LFS-F05133?style=flat-square&logo=gitlfs&logoColor=white)](#)

下面按 **「必做 → 选做」** 写，避免读起来前后打架：**必做** 是克隆本仓库后想正常打开页面、播视频就离不开的步骤；**选做** 只有在你明确要用云端、真地图、真推送时才需要，不做也能做界面演示和录屏。

**环境与命令说明**：全文以 **Windows 10 / Windows 11** 为准；命令请在 **PowerShell**、**命令提示符（cmd）** 或安装 [Git for Windows](https://git-scm.com/download/win) 后自带的 **Git Bash** 中执行（任选其一即可）。

---

## 1. 项目简介

**StarTrack·星途** 是一款面向演唱会与大型会展（MICE）场景的 **移动端 Web 单页应用（SPA）** 原型，演示从 **活动发现**、**行程与路线**、**现场地图与安全**、**粉丝社区与内容**、**出行协作** 到 **个人中心** 的完整动线。入口是仓库根目录下的 **`index.html`**。下文 **「仓库根目录」** 指克隆后的项目根路径（与 `index.html` 同级）。

样式与图标来自 **Tailwind CSS**、**Font Awesome** 的 **CDN**，本机需要能访问公网。

### 1.1 必做 vs 选做（先看这张表）

| 类别 | 要做什么 | 不做会怎样 |
| :--- | :--- | :--- |
| **必做** | 安装 **Git**、**Git LFS** 后，按 **第 4.2 节** **从 `git --version` 到 `git lfs pull` 逐条执行** | `index.html` 或 **`会展地图高保真/` 里的 mp4 等视频** 可能仍是 **LFS 指针**，页面空白、视频黑屏或无法播放。本仓库里 **上传的一批 mp4 等媒体文件走 Git LFS**，不拉 LFS 就**没有**真实二进制。 |
| **选做：高德 Web 地图** | 在 **`index.html`** 填写有效的 **`STARTRACK_AMAP_KEY`**（见 [第 5.2 节](#52-高德地图开放平台与-key)） | 页内 **嵌入式地图**（`AMap`）不加载；**不影响**多数页面点选、文案、本地会话。 |
| **选做：Firebase** | 按 [第 5 节](#5-配置指南) 配置 **`firebaseConfig`**（`index.html` 与 **`firebase-messaging-sw.js`** 两处一致）并开通 Firestore / FCM 等 | **Firestore 写入/读取、FCM 推送** 不可用；控制台常见初始化报错；**多数纯前端界面仍可浏览、录屏**。 |
| **选做：本地 HTTP** | 用 Node / Python 等在仓库根起 **`localhost`** 或 **HTTPS** 静态服务（见 [第 4.3 节](#43-在仓库根目录启动本地-http-服务)） | 仅用 **`file://` 双击打开** 时，**Service Worker 不会注册**，**FCM 无法按完整链路测试**；其余与是否配 Firebase / 高德独立。 |

**和「本机装没装软件」的关系**：不需要在电脑上安装「Firebase 客户端」或「高德 PC 版」才能打开网页；Firebase / 高德 **Web SDK 由浏览器从 CDN 加载**。Key 是写在文件里的**字符串**，不是系统驱动。

**导航按钮**：部分「开始导航」等使用 **`amapuri://`** 尝试唤起**手机高德 App**；PC 无对应客户端时常见无法打开链接，与 Firebase 无关。页内地图是否显示仍取决于 **高德 Web Key**（见上表「选做：高德」）。

**密钥**：真实 `apiKey`、高德 Key **不要** push 到公开仓库。

---

## 2. 核心功能

*   **账号与会话**：登录、注册、实名认证；**`localStorage`**（如 `startrack.session.v1`）与 **`window.pageHistory`** 做会话与返回导航；无会话时多从 **登录栈** 进入。
*   **首页与活动发现**：活动卡片、专题、搜索、收藏；可进入行程、应援一日游、当地探索、打卡/攻略、粉丝社区、已生成路线等。
*   **行程、路线与活动详情**：行程页、自定义路线、路线结果与详情；活动详情、接驳/班车、专题页；地图区块依赖 **高德 JS API + 有效 Key**（[第 5.2 节](#52-高德地图开放平台与-key)）。
*   **现场、直播与安全**：直播页、现场地图、推送与陪伴设置；安全热力、安全中心、告警、举报等；部分设计与 **Firestore** 联动（需完成 Firebase 选做配置）。
*   **社区与 UGC**：粉丝社区、攻略发布与详情、观演评价生成与预览。
*   **出行与协作（原型）**：拼车、拼房、群聊等为页面级原型，实时能力需另接后端。
*   **个人中心与设置**：资料、收藏、物品清单、隐私与通用设置、帮助与关于。
*   **工程体验**：图片懒加载（**`IntersectionObserver`** + 降级）、**`toggleCard`** 卡片展开收起、全站搜索（繁简归一、模糊匹配与索引）。

---

## 3. 技术架构

| 模块 | 技术方案 |
| :--- | :--- |
| **形态** | 单页应用（SPA）：`div` 显隐切换；**`window.showPage`** 路由式跳转 |
| **界面与样式** | HTML5 + **Tailwind CSS**（CDN `cdn.tailwindcss.com`；注释提示生产可用 PostCSS/CLI） |
| **图标** | **Font Awesome 4.7**（CDN） |
| **脚本** | 原生 **JavaScript（ES6+）**；主逻辑在 **`index.html`** 内联与事件绑定 |
| **云端数据** | **Firebase 9.23.0 兼容模式**：`firebase-app-compat`、`firebase-firestore-compat` |
| **消息推送** | **FCM** 兼容模式 + **`firebase-messaging-sw.js`**（`importScripts` 加载模块化 Firebase **9.23.0**） |
| **地图** | **高德 JS API 2.0**（Key 通过前端校验后才插入 `<script>`） |
| **本地持久化** | **`localStorage`** |
| **大文件与版本库** | **Git LFS**（**`会展地图高保真/` 下大量 mp4 等视频由 LFS 存储**；另见根目录 **`.gitattributes`**，是否包含其他路径以文件为准） |

---

## 4. 安装指南

### 4.1 环境与工具

*   **操作系统**：**Windows 10 或 Windows 11**（下文命令均在此环境下验证思路编写）。
*   **浏览器**：推荐 **Microsoft Edge** 或 **Google Chrome**（Windows 版），便于打开 **开发者工具** 查看 **Console** 与 **Service Worker**。
*   **Git for Windows**：[下载安装](https://git-scm.com/download/win)，安装时若勾选 **Git LFS** 可少装一步；未勾选则须再装 **Git LFS**（见下节）。
*   **Git LFS**（**必装**）：本仓库 **视频等媒体（含 `会展地图高保真/` 下 mp4）通过 LFS 上传**，无 LFS 或未完成拉取则无法得到可播放文件。完整命令见 **第 4.2 节**。
*   **可选 Node.js（Windows 安装包）**：用于 [第 4.3 节](#43-在仓库根目录启动本地-http-服务) 中的 `npx`（[官网下载](https://nodejs.org/)）。
*   **可选 Python 3（Windows 安装包）**：用于同节中的 `python` / `py` 启动简易 HTTP 服务（[官网下载](https://www.python.org/downloads/windows/)）。
*   **网络**：需能访问 Tailwind、Font Awesome、Firebase、高德等 **CDN**。

### 4.2 Git 与 Git LFS：完整命令步骤

**原因**：`.gitattributes` 将 **`会展地图高保真/` 下 mp4 等** 以及可能的其他大文件标为 **LFS**。克隆后仓库里先是 **文本指针**；执行 **`git lfs pull`** 后才会在本机出现 **真实 mp4 二进制**，页面里的 `<video src="./会展地图高保真/...">` 才能播。若 **`index.html` 等也被列入 LFS**（以 `.gitattributes` 为准），未拉取时同样可能几乎空白。

以下命令均在 **PowerShell / cmd / Git Bash** 中逐条输入；**每行是一条完整命令**，回车执行后再输入下一条（方括号里是说明，**不要**把方括号打进命令里）。

*   **Windows**：官网安装包，装完**重开终端**。
*   **macOS**：`brew install git-lfs`
*   **Linux（Debian/Ubuntu）**：`sudo apt install git-lfs`

**（一）先装好 Git LFS 程序本体（任选一种，只做一次）**

*   **方式 A**：打开 [git-lfs.com](https://git-lfs.com/)，下载 **Windows `.exe` 安装包**，双击安装。  
*   **方式 B**：在 **PowerShell** 执行：`winget install GitHub.GitLFS`  
*   **方式 C**：安装 [Git for Windows](https://git-scm.com/download/win) 时勾选 **Git LFS** 组件。

装完后**关掉窗口再新开** PowerShell / cmd / Git Bash，再继续下面「（二）」。

---

**（二）确认 Git 与 Git LFS 能在命令行里被调用**

```bash
git --version
```

```bash
git lfs version
```

若第二条报错「不是内部或外部命令」，说明 **Git LFS 未装好或未进 PATH**，请回到（一）重装后再试。

---

**（三）在本机启用 Git LFS 钩子（每台 Windows 用户建议执行一次）**

```bash
git lfs install
```

成功时一般会看到 `Git LFS initialized` 一类提示。

---

**（四）克隆远程仓库到本机**

把下面命令里的 `https://github.com/你的用户名/你的仓库名.git` 换成你在 GitHub 网页上 **Code → HTTPS**（或 SSH）里**复制下来的整段地址**（不要带尖括号）：

```bash
git clone https://github.com/你的用户名/你的仓库名.git
```

执行结束后，当前目录下会出现一个与仓库同名的文件夹。

---

**（五）进入仓库根目录**

把 `你的仓库名` 换成上一步 **`git clone` 实际生成的文件夹名**（与 GitHub 仓库名通常一致；若在别的盘，可写绝对路径，例如 `cd D:\CodingProjects\你的仓库名`）：

```bash
cd 你的仓库名
```

可选：确认已在正确目录（应能看到 `index.html`；**不是 Git 命令**，仅 Windows 下列目录用）：

在 **cmd / PowerShell** 中：

```bash
dir
```

在 **Git Bash** 中：

```bash
ls
```

---

**（六）拉取 LFS 大文件（必做，否则 mp4 等可能无法播放）**

仍在**仓库根目录**下执行：

```bash
git lfs pull
```

若提示已是最新或没有需要下载的，一般也说明已拉过；若之前拉失败，可再试一次下面两条（先 fetch 再检出，与 `pull` 二选一或作为补救）：

```bash
git lfs fetch --all
```

```bash
git lfs checkout
```

---

**（七）可选：查看当前仓库里哪些路径由 LFS 管理**

```bash
git lfs ls-files
```

---

**（八）可选：查看工作区是否干净、是否在正确分支**

```bash
git status
```

---

**自检（不用敲命令，用资源管理器或记事本看即可）**

*   **`会展地图高保真/`** 下应有 **体积正常** 的 `.mp4`；若文件极小或打不开，多半是 **（六）** 未成功。  
*   **`index.html`** 应为大段 HTML；若以 `version https://git-lfs.github.com/spec/v1` 开头，仍是 **LFS 指针**，请回到 **（六）** 重试，并确认 **（二）** 里 `git lfs version` 正常。

### 4.3 在仓库根目录启动本地 HTTP 服务（Windows）

**用途**：在 **`http://localhost`** 或 **HTTPS** 下才能注册 **`firebase-messaging-sw.js`**，从而完整测试 **FCM**；**`file://`** 下代码会 **跳过** Service Worker。

先用 **`cd`** 进到 **仓库根目录**（该目录下应能直接看到 `index.html` 与 `firebase-messaging-sw.js`）。可在资源管理器中打开该文件夹，在地址栏输入 **`powershell`** 回车，即在此目录打开 PowerShell。

**方式一：已安装 Node.js 时**

在 **PowerShell / cmd / Git Bash** 中执行：

```bash
npx --yes serve .
```

窗口里会打印 **`http://localhost:某端口`**，用本机浏览器打开即可。

**方式二：已安装 Python 3 时**

在同一目录下执行（若 `python` 无效，可改用 Windows 自带的启动器 **`py`**）：

```bash
python -m http.server 8080
```

或：

```bash
py -m http.server 8080
```

浏览器访问 `http://localhost:8080/`（端口与命令中一致）。

**方式三：Visual Studio Code（Windows）**  
安装 **Live Server** 扩展，在 VS Code 中打开仓库根目录，对 `index.html` 使用 **「Open with Live Server」**，按扩展提示的 **`http://127.0.0.1:端口`** 访问；同样需保证站点根即仓库根，且能访问到 **`firebase-messaging-sw.js`**。

### 4.4 本地文件协议与 HTTP

*   **`file://` 双击 `index.html`**：最省事；**不注册** Service Worker，**FCM 不测全**；Firebase 仍会尝试初始化，占位时控制台有报错属正常。
*   **本地 HTTP**：便于测 **Service Worker、通知、FCM token**；需在 Windows 上按 **第 4.3 节** 安装 **Node.js** 或 **Python 3**（或使用 VS Code Live Server）之一。

### 4.5 相对路径与 `会展地图高保真/`

*   页面引用 **`./会展地图高保真/...` 下的 mp4 等**；请保持该目录与 **`index.html`** 的相对位置不变，否则 **404**。
*   **mp4 本体由 Git LFS 管理**：除克隆外务必按 **第 4.2 节** 执行 **`git lfs pull`**（及前文所列 Git 命令）。

---

## 5. 配置指南

> **整章为选做。** 仅当你需要 **Firestore 真读写**、**FCM 真推送**、或 **页内高德地图稳定显示** 时，从本节开始操作。不需要这些能力时，**整章可跳过**；占位 Firebase 时控制台可能报错，与上文 **「1.1 必做 vs 选做」** 那张表一致。

### 5.1 Firebase 控制台与 Web 应用

**5.1.1 创建项目**  
[Firebase Console](https://console.firebase.google.com/) → 添加项目（可按需关闭 Analytics）。

**5.1.2 注册 Web 应用**  
添加 **Web** 应用，记录 **`firebaseConfig`**：`apiKey`、`authDomain`、`projectId`、`storageBucket`、`messagingSenderId`、`appId`。

**5.1.3 启用 Cloud Firestore**  
「构建」→「Firestore Database」建库；课设可暂用 **测试模式**（注意 **规则过期**）；长期公开请收紧规则（[第 5.4 节](#54-cloud-firestore-安全规则)）。

**5.1.4 启用 Cloud Messaging（FCM）**  
「构建」→「Cloud Messaging」；若要求 **Web 推送证书 / 密钥对**，按控制台与文档完成，否则 **`messaging.getToken`** 可能失败。

### 5.2 高德地图开放平台与 Key

**5.2.1 申请 Key**  
[高德开放平台](https://lbs.amap.com/) →「应用管理」→「我的应用」→ 添加 **Key**，类型选 **「Web 端（JS API）」**，与项目 **JS API 2.0** 一致。

**5.2.2 安全密钥**  
若 Key 启用 **jscode** 等，按 [高德 JS API 2.0 安全说明](https://lbs.amap.com/api/javascript-api-v2/guide/abc/load) 配置（如 **`window._AMapSecurityConfig`**）。仓库默认仅在脚本 URL 传 **`key`**；鉴权失败先查文档，勿乱改媒体相对路径。

**5.2.3 写入 `index.html`**

```javascript
window.STARTRACK_AMAP_KEY = '您的高德地图AK密钥';
```

替换为你的 **Web Key**（无中文占位、无多余空格）。

**5.2.4 内置校验**  
空 Key、过短、含中文或占位词时 **不请求** 高德脚本，避免 **Error key** 刷屏。配置正确后刷新，依赖地图页可出现 **`AMap`**。

**5.2.5 上线**  
在控制台为 Key 配置 **HTTP Referer** 等防盗用。

### 5.3 将 Firebase 配置写入两处（必须一致）

| 文件 | 说明 |
| :--- | :--- |
| **`index.html`** | **compat** 初始化 `firebase-app`、`firestore`、`messaging` |
| **`firebase-messaging-sw.js`** | **`importScripts`** 加载模块化 **`firebase-app.js` / `firebase-messaging.js`（9.23.0）**，再 **`firebase.initializeApp(firebaseConfig)`** |

1. **`index.html`** 中搜索 **`const firebaseConfig = {`**，整段替换。  
2. **`firebase-messaging-sw.js`** 顶部 **`firebaseConfig`** 使用**同一份**复制。  
3. **localhost 或 HTTPS** 打开，看 **Console** 是否初始化成功；检查 JSON 语法。

勿将真实配置提交**公开**远程仓库。

### 5.4 Cloud Firestore 安全规则

存在 **`firebase.firestore.FieldValue.serverTimestamp()`** 等写入；规则拒绝时控制台 **Permission denied**。开发可用测试规则并注意过期；生产按集合编写 **`allow read, write`**。集合名在 **`index.html`** 搜 **`db.collection`**。改规则后 **发布**。

### 5.5 FCM 与 Service Worker

*   **`navigator.serviceWorker.register('firebase-messaging-sw.js')`**：默认与 **`index.html`** 同根路径；部署到子路径时需自行调整。  
*   通知引用 **`icon.png`**（相对 SW）；缺失则图标可能异常，可自备 **`icon.png`**。  
*   **通知权限**被拒则无 **FCM token**，属预期。

---

## 6. 使用说明（文字）

无配图；可边开浏览器边对照。

*   **6.1 首次与会话**：读 **`localStorage`**；无有效会话则从 **登录栈** 起；登录后多进 **首页**，**`showPage('…')`** 切换区块。  
*   **6.2 登录 / 注册 / 实名**：独立区块，**`showPage`** 切换；校验规则以**代码**为准。  
*   **6.3 首页、搜索、收藏**：搜索用全站索引；收藏是否落 Firestore 取决于是否完成 Firebase 选做配置。  
*   **6.4 行程与活动**：个人中心快捷入口可进行程、一日游、探索、攻略、社区、路线管理；地图依赖高德脚本；**`showPage`** + **`pageHistory`** 管返回。  
*   **6.5 现场与推送**：HTTP + Firebase + 通知权限下可观察 **FCM token** 与控制台发测。  
*   **6.6 社区与评价**：社区、攻略、观演评价流程演示。  
*   **6.7 拼车与群聊**：原型级，无独立实时后端。  
*   **6.8 个人中心与设置**：资料、设置、帮助等；**`.back-button`** 无单独 `id` 时依 **`pageHistory`** 回退。

---

## 7. 常见问题与排查

| 现象 | 可能原因 | 建议处理 |
| :--- | :--- | :--- |
| 克隆后 `index.html` 几乎为空 | **未拉 Git LFS**（若该文件被 LFS 跟踪） | `git lfs install` → 仓库内 **`git lfs pull`** |
| **mp4 黑屏、无法播放** | **`会展地图高保真/` 下视频仍为 LFS 指针**，或目录相对位置被移动 | **`git lfs pull`**；保持 **`./会展地图高保真/`** 与 `index.html` 相对路径 |
| 控制台 Firebase 初始化失败 | **`firebaseConfig` 错误** 或服务未开 | 核对 Firestore/FCM；两处配置一致 |
| Service Worker 失败 | **`file://`** 或非安全上下文 | 换 **`http://localhost`** 或 **HTTPS**；确认存在 **`firebase-messaging-sw.js`** |
| 无 `AMap` | **Key 未过校验** 或需 **安全密钥** | **`STARTRACK_AMAP_KEY`**；按高德文档补 **jscode** |
| 无 FCM token | **通知被拒** 或 **SW 未注册** | 允许通知；用 HTTP(s) |
| Firestore 写入失败 | **安全规则** | [第 5.4 节](#54-cloud-firestore-安全规则) |
| 样式丢失 | **CDN 不可达** | 检查网络与拦截 |
| 「开始导航」异常 | **`amapuri://` 唤起高德 App**；与页内 **`AMap`** 无关 | 手机装高德试；PC 无客户端常见 |

---

## 8. 仓库结构（与 GitHub 根目录一致）

```
（仓库根目录）
├── index.html                      # 主应用入口（SPA）
├── firebase-messaging-sw.js        # FCM Service Worker（firebaseConfig 须与 index.html 一致）
├── .gitattributes                  # Git LFS 规则（含会展地图高保真下 mp4 等，以文件为准）
├── LICENSE                         # 若存在
├── README.md                       # 本说明
└── 会展地图高保真/                  # 页面引用 ./会展地图高保真/...（mp4 等一般由 LFS 存储）
    ├── 个人/
    ├── 组合/
    └── …
```

---

## 9. 声明

本项目用于 **课程或原型演示**。使用 **Firebase、高德** 须遵守各平台协议与配额；**密钥泄露与规则误配** 的风险由使用者承担。**请勿**将生产密钥推送到公共托管平台。

---

**迷路时**：先回到 **「1.1 必做 vs 选做」** 那张表，分清当前是 **必做** 还是 **选做**，再查 **第 7 节** 常见问题表。
