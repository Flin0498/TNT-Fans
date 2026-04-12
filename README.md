# StarTrack·星途（MICE1901）

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)](#)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)](#)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](#)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore%20%7C%20FCM-FFCA28?style=flat-square&logo=firebase&logoColor=black)](#)
[![Git LFS](https://img.shields.io/badge/Git-LFS-F05133?style=flat-square&logo=gitlfs&logoColor=white)](#)

---

## 1. 项目简介

**StarTrack·星途** 是一款面向演唱会与大型会展（MICE）场景的 **移动端 Web 单页应用（SPA）** 原型，用于演示「活动发现—行程与路线—现场地图与安全—粉丝社区与内容—出行协作—个人中心」等完整动线。应用主入口为仓库根目录下的 **`index.html`**，界面样式依赖 **Tailwind CSS**、图标依赖 **Font Awesome** 等 **CDN** 资源，因此运行设备需要能访问公网以加载这些静态资源。

本仓库在 **GitHub 上克隆后没有名为「定稿」的父级文件夹**：远程仓库根目录即为 **`index.html`、`firebase-messaging-sw.js`、`.gitattributes`、`会展地图高保真/`** 等与网页列表一致的结构。若你本地开发时习惯使用「外层文件夹 + 内层 Git 根目录」的嵌套布局，凡本文出现「仓库根目录」均指 **含有 `.git` 与 `index.html` 的那一层**，请勿与本地外层目录混淆。

**重要限制（务必读完）**：

*   **Git LFS**：部分大文件由 Git LFS 管理。未正确安装并拉取 LFS 时，你看到的 `index.html` 等可能仅为几行指针文件，**无法在浏览器中正常打开应用**。详见 [第 4.2 节](#42-git-lfs-安装与拉取)。
*   **Firebase 与高德**：源码中带占位配置。不替换为你的真实项目配置时，**云端数据库、推送、地图等能力不可用或表现为控制台报错**；这不属于「路径错误」，而是**必须完成的配置**。详见 [第 5 节](#5-配置指南)。
*   **`file://` 打开方式**：双击 `index.html` 使用本地文件协议时，项目代码会 **主动跳过 Service Worker 注册**，因此 **无法完整验证 FCM 推送**；地图与 Firebase 仍取决于你是否完成配置。详见 [第 4.4 节](#44-本地文件协议与-http-的区别)。
*   **本文不包含** 使用仓库内任何 **PowerShell 一键启动脚本** 的说明；本地 HTTP 请使用 **Node、`python -m http.server`、编辑器 Live Server、Caddy、Nginx** 等通用静态服务方式。
*   **密钥安全**：切勿将含真实 `apiKey`、高德 Key 的修改推送到**公开**仓库。

---

## 2. 核心功能

*   **账号与会话**
    *   提供登录、注册、实名认证等相关页面流程。
    *   使用 **`localStorage`**（如键 `startrack.session.v1`）保存本地会话信息（例如手机号）；与 **`window.pageHistory`** 配合，实现多页栈式返回，减少「无路可退」的导航死胡同。
    *   未检测到有效会话时，应用倾向于从 **登录栈** 进入；会话恢复后进入 **首页栈**。

*   **首页与活动发现**
    *   活动卡片、演唱会专题入口、搜索入口与收藏入口。
    *   支持从首页进入 **行程、应援一日游、当地探索、打卡/攻略、粉丝社区、已生成路线管理** 等模块。

*   **行程、路线与活动详情**
    *   **行程页、自定义路线、路线结果、路线详情** 等页面联动。
    *   **活动详情、接驳/班车详情、专题演唱会页** 等展示与跳转。
    *   地图相关区块依赖 **高德 JS API** 与有效 Key（见 [第 5.2 节](#52-高德地图开放平台与-key))。

*   **现场、直播与安全**
    *   **直播相关页、现场地图、直播推送设置、陪伴设置** 等。
    *   **安全热力、安全中心、告警详情、举报表单** 等与安全相关的信息架构。
    *   部分演示数据或实时数据设计为与 **Firestore** 集合联动（需启用服务并配置规则）。

*   **社区与 UGC**
    *   **粉丝社区、发布攻略、攻略列表与详情、观演评价生成与预览** 等。

*   **出行与协作（原型级）**
    *   **拼车、拼房、群组聊天、群聊房间** 等页面与交互占位，用于展示信息架构与跳转关系。

*   **个人中心与系统设置**
    *   **个人资料、收藏、物品清单与编辑、隐私设置、通用设置、帮助与反馈、关于** 等。

*   **工程与体验增强**
    *   **图片懒加载**：优先使用 **`IntersectionObserver`**；不支持时回退为直接赋值 `src`。
    *   **卡片展开/收起**：通过 `toggleCard` 等函数切换显隐与图标旋转状态。
    *   **全站搜索**：内置繁简归一化、模糊匹配与索引逻辑，供各搜索栏与搜索页复用。

---

## 3. 技术架构

| 模块 | 技术方案 |
| :--- | :--- |
| **形态** | 单页应用（SPA）：大量页面区块以 `div` 的显隐（如 `hidden`）切换；统一通过 **`window.showPage`** 进行路由式跳转 |
| **界面与样式** | HTML5 + **Tailwind CSS**（通过 CDN `cdn.tailwindcss.com` 引入；注释中提示生产环境可使用 PostCSS/CLI 构建） |
| **图标** | **Font Awesome 4.7**（CDN） |
| **脚本** | 原生 **JavaScript（ES6+）**；主要逻辑位于 **`index.html`** 内联脚本及页面内事件绑定 |
| **云端数据** | **Firebase 9.23.0 兼容模式**：`firebase-app-compat`、`firebase-firestore-compat` |
| **消息推送** | **Firebase Cloud Messaging** 兼容模式 + 根目录 **`firebase-messaging-sw.js`**（Service Worker 内使用 `importScripts` 加载 **模块化** Firebase 9.23.0 脚本） |
| **地图** | **高德地图 JavaScript API 2.0**（仅在 Key 通过前端校验时动态插入 `<script>`） |
| **本地持久化** | **`localStorage`**（会话、部分偏好） |
| **大文件与版本库** | **Git LFS**（规则见根目录 **`.gitattributes`**；具体跟踪路径以该文件为准） |

---

## 4. 安装指南

### 4.1 环境与工具

*   **操作系统**：Windows、macOS、Linux 均可。
*   **浏览器**：建议使用 **Google Chrome** 或 **Microsoft Edge（Chromium 内核）**，便于查看 **开发者工具 Console** 与调试 **Service Worker / Application** 面板。
*   **Git**：用于克隆与更新仓库，[官方下载](https://git-scm.com/downloads)。
*   **Git LFS**：本仓库依赖 **Git Large File Storage**，[官网](https://git-lfs.com/)。
*   **可选 Node.js**：用于执行 `npx serve` 等命令。
*   **可选 Python 3**：用于执行 `python -m http.server`。
*   **网络**：首次打开页面需能访问 **Tailwind / Font Awesome / Firebase / 高德 CDN** 等外链。

### 4.2 Git LFS 安装与拉取

**为什么要做这一步**  
若 `.gitattributes` 将 `index.html`、`firebase-messaging-sw.js` 或 `会展地图高保真/` 等纳入 LFS，则 Git 仓库中保存的是 **文本指针**；只有执行 LFS 拉取后，才会替换为 **真实文件内容**。跳过此步的常见后果包括：`index.html` 用浏览器打开几乎空白、视频无法播放、Service Worker 脚本异常等。

**安装 Git LFS（择一平台）**

*   **Windows**：从 Git LFS 官网下载安装程序并安装；安装完成后重新打开终端。
*   **macOS**（已安装 Homebrew）：`brew install git-lfs`
*   **Linux（Debian/Ubuntu 示例）**：`sudo apt install git-lfs`

**全局启用（每台计算机建议执行一次）**

```bash
git lfs install
```

**克隆仓库**

```bash
git clone <你的仓库 HTTPS 或 SSH 地址>
cd <仓库目录名>
```

**拉取大文件内容**

```bash
git lfs pull
```

**如何自检是否成功**

*   用文本编辑器打开 **`index.html`**：若能看到大量 HTML 与脚本内容、文件体积显著大于几 KB，通常表示已拉取成功。
*   若文件开头为 `version https://git-lfs.github.com/spec/v1` 等字样，说明仍是 **LFS 指针**，请检查是否已安装 LFS、是否在仓库目录执行了 `git lfs pull`，或联系维护者确认远程 LFS 存储是否可用。

### 4.3 在仓库根目录启动本地 HTTP 服务

**为什么要用 HTTP**  
在 **`http://localhost`** 或 **HTTPS** 下访问时，浏览器才处于 **安全上下文**，项目中的 **`navigator.serviceWorker.register('firebase-messaging-sw.js')`** 才可能成功；**`file://` 协议下代码会跳过注册**，无法完整验证推送链路。

**操作前提**：终端当前工作目录必须为 **仓库根目录**（该目录下应能直接看到 `index.html` 与 `firebase-messaging-sw.js`）。

**方式一：Node（推荐）**

```bash
npx --yes serve .
```

终端会打印形如 `http://localhost:3000` 的地址，用浏览器打开即可（具体端口以终端输出为准）。

**方式二：Python 3**

```bash
python -m http.server 8080
```

浏览器访问 `http://localhost:8080/`（端口可自定义，与命令一致即可）。

**方式三：其他工具**  
任意能将 **仓库根目录** 作为站点根目录提供静态文件访问的方式均可，例如 **VS Code Live Server**、**Caddy**、**Nginx** 等。原则不变：**同源**下能访问到 **`index.html`** 与 **`firebase-messaging-sw.js`**，且使用 **localhost 或 HTTPS**。

### 4.4 本地文件协议与 HTTP 的区别

*   **仅双击打开 `index.html`（`file://`）**  
    *   **优点**：零命令行，最快看到静态界面。  
    *   **缺点**：控制台会出现说明——**不会注册 Service Worker**，**FCM 推送无法按设计完整验证**；部分浏览器对本地文件的跨域与能力限制更严格。  
    *   **Firebase 初始化**：仍可能执行，但取决于网络与配置是否正确；若配置为占位符，控制台会有初始化失败等日志。

*   **通过本地 HTTP 访问**  
    *   **优点**：与真实部署更接近，可验证 **Service Worker**、**通知权限**、**FCM token** 等。  
    *   **缺点**：需要安装 Node/Python 或其他静态服务器之一。

### 4.5 相对路径与资源目录

*   页面中视频等媒体使用 **`./会展地图高保真/...`** 等 **相对路径**。请 **不要随意移动** `index.html` 与 `会展地图高保真` 的相对位置；否则会出现 **资源 404**。
*   若你从 GitHub 克隆得到的就是「根目录即项目」，则 **无需改任何路径**；只需保证 LFS 资源已拉取完整。

---

## 5. 配置指南

### 5.1 Firebase 控制台与 Web 应用

**5.1.1 创建项目**  
打开 [Firebase Console](https://console.firebase.google.com/)，登录后「添加项目」，按向导完成创建（演示环境可按需关闭 Google Analytics）。

**5.1.2 注册 Web 应用**  
在项目概览中选择 **「</>」** 添加 **Web** 应用，记录向导给出的 **`firebaseConfig`** 字段：`apiKey`、`authDomain`、`projectId`、`storageBucket`、`messagingSenderId`、`appId`。

**5.1.3 启用 Cloud Firestore**  
在控制台 **「构建」→「Firestore Database」** 中创建数据库。课程或本地调试可临时使用 **测试模式**（注意控制台提示的 **规则过期时间**）；长期或对外演示请改为受限规则（见 [第 5.4 节](#54-cloud-firestore-安全规则))。

**5.1.4 启用 Cloud Messaging（FCM）**  
在 **「构建」→「Cloud Messaging」** 中按界面指引操作。若控制台要求配置 **Web 推送证书 / 密钥对** 等，请按官方文档完成，否则部分环境下 **`messaging.getToken`** 可能失败。

### 5.2 高德地图开放平台与 Key

**5.2.1 申请 Key**  
访问 [高德开放平台](https://lbs.amap.com/)，完成注册登录后，在 **「应用管理」→「我的应用」** 中创建应用，并添加 **Key**；服务平台请选择 **「Web 端（JS API）」**，以匹配项目中加载的 **JS API 2.0**。

**5.2.2 安全密钥（如控制台要求）**  
若你的 Key 启用了 **安全密钥（jscode）** 等策略，需遵循 [高德 JS API 2.0 加载与安全说明](https://lbs.amap.com/api/javascript-api-v2/guide/abc/load) 在页面中正确配置（例如部分场景需在加载地图脚本前设置 **`window._AMapSecurityConfig`**）。**当前仓库默认仅在脚本 URL 中传入 `key` 参数**；若遇鉴权错误，请对照高德文档补齐，而非盲目修改相对路径。

**5.2.3 写入 `index.html`**  
在文件靠前位置找到：

```javascript
window.STARTRACK_AMAP_KEY = '您的高德地图AK密钥';
```

将引号内替换为你的 **Web 端 Key**（字符串中勿留中文占位、勿含空格）。

**5.2.4 内置校验逻辑（无需改路径）**  
若 Key 为空、过短、包含中文、或命中「请替换」「密钥」等占位关键词，项目 **不会** 请求高德脚本，以避免控制台出现大量 **Error key** 类报错。配置正确后刷新页面，在依赖地图的页面中应能观察到 **`typeof AMap !== 'undefined'`** 相关逻辑生效。

**5.2.5 正式上线**  
在开放平台为 Key 配置 **HTTP Referer** 等限制，防止 Key 被第三方网站盗用。

### 5.3 将 Firebase 配置写入两处代码（必须一致）

本项目存在 **两套** Firebase 初始化入口，**配置对象必须逐字段一致**：

| 文件 | 说明 |
| :--- | :--- |
| **`index.html`** | 使用 **compat** 脚本初始化 `firebase-app`、`firestore`、`messaging` |
| **`firebase-messaging-sw.js`** | Service Worker 内通过 **`importScripts`** 加载 **模块化** `firebase-app.js` 与 `firebase-messaging.js`（版本 **9.23.0**），再调用 **`firebase.initializeApp(firebaseConfig)`** |

**操作步骤**

1. 在 **`index.html`** 中搜索 **`const firebaseConfig = {`**，用你在控制台获得的配置 **完整替换** 占位对象。  
2. 打开 **`firebase-messaging-sw.js`**，对其顶部的 **`firebaseConfig`** 做 **同样修改**（建议复制粘贴同一份，避免手误）。  
3. 保存后，在 **localhost 或 HTTPS** 下打开站点，打开 **Console**：应看到类似 **Firebase 初始化成功** 的日志；若失败，检查 JSON 语法（英文引号、逗号、无注释混入等）。

**切勿**将真实配置提交到公开远程仓库；可使用私有分支、本地忽略策略或改为构建时注入（需自行改造工程）。

### 5.4 Cloud Firestore 安全规则

代码中存在向 Firestore **写入** 数据（例如使用 **`firebase.firestore.FieldValue.serverTimestamp()`**）的逻辑。若规则为 **全部拒绝**，浏览器控制台会出现 **Permission denied** 等错误。

*   **开发期**：可使用控制台提供的测试规则，但务必留意 **过期时间**。  
*   **演示/生产**：在 **Firestore → 规则** 中按集合路径编写 **`allow read, write`** 条件，避免长期 **`if true`** 全开。

具体集合名称请在 **`index.html`** 中搜索 **`db.collection`** 自行核对；修改规则后需点击 **发布** 生效。

### 5.5 FCM 与 Service Worker 行为摘要

*   注册语句为 **`navigator.serviceWorker.register('firebase-messaging-sw.js')`**，因此 **SW 文件默认与 `index.html` 位于同一根路径**。若部署到 **子路径**（如 `https://example.com/app/`），需同步调整注册 URL 与部署结构（属进阶话题）。  
*   **`firebase-messaging-sw.js`** 中通知使用了 **`icon.png`**（相对 Service Worker URL 解析）。若根目录无该文件，通知仍可能出现，但图标可能异常；可自行添加 **`icon.png`**。  
*   页面加载后会请求 **浏览器通知权限**；用户拒绝时无法拿到 **FCM token**，属预期行为。

---

## 6. 使用说明（文字）

以下按典型使用顺序描述界面与操作逻辑，**不附带截图**；你可自行在浏览器中对照。

### 6.1 首次进入与会话恢复

*   页面加载时会尝试读取 **`localStorage`** 中的会话数据；若不存在有效信息，**页面历史栈** 会从 **登录** 相关状态开始。  
*   完成登录流程后，通常可进入 **首页**，底部或页面内的导航入口会调用 **`showPage('…')`** 切换到对应区块。

### 6.2 登录、注册与实名认证

*   **登录页、注册页、实名认证页** 为独立区块，通过 **`showPage`** 切换显示。  
*   密码或表单校验规则以页面内实现为准（若与课程要求不一致，以代码为准）。

### 6.3 首页与搜索、收藏

*   **首页** 聚合活动入口、专题卡片与快捷操作。  
*   **搜索** 会跳转到搜索相关页面，并复用全站搜索索引逻辑。  
*   **收藏** 跳转到收藏页，用于管理用户标记的内容（具体数据是否落 Firestore 取决于你是否启用云端与规则）。

### 6.4 行程、路线与活动详情

*   从首页或 **个人中心快捷入口** 可进入 **行程、应援一日游、当地探索、打卡/攻略、社区、已生成路线** 等。  
*   **行程页** 可承载演唱会/活动上下文；**地图类组件** 依赖高德脚本是否成功加载。  
*   **活动详情、接驳详情、安全热力** 等子页通过 **`showPage`** 与 **`pageHistory`** 维护返回路径。

### 6.5 现场、直播与推送相关设置

*   **直播页、现场地图页、直播推送设置、陪伴设置** 用于展示现场动线与通知偏好。  
*   在 **HTTP + 有效 Firebase 配置 + 用户同意通知** 的前提下，控制台可打印 **FCM token**；可在 Firebase 控制台尝试发送测试消息（界面随官方更新而变化，此处不逐步截图）。

### 6.6 社区、攻略与评价

*   **粉丝社区、发布攻略、攻略详情** 等页面展示 UGC 流程。  
*   **观演评价生成与预览** 用于演示从输入到预览的步骤型交互。

### 6.7 拼车、拼房与群聊（原型）

*   **拼车、拼房、群组聊天、群聊房间** 以页面与按钮为主，展示信息架构；深度实时通信需额外后端，不在本静态原型范围内。

### 6.8 个人中心与设置

*   **个人资料** 可编辑头像、昵称等信息（以页面实现为准）。  
*   **设置** 中包含隐私、推送、关于、帮助与反馈等入口。  
*   **返回键**：未单独绑定 `id` 的 **`.back-button`** 会依赖 **`window.pageHistory`** 进行统一回退。

---

## 7. 常见问题与排查

| 现象 | 可能原因 | 建议处理 |
| :--- | :--- | :--- |
| 克隆后 `index.html` 几乎为空 | **未拉取 Git LFS** | 安装 LFS → `git lfs install` → 在仓库内 `git lfs pull` |
| 视频黑屏或无法播放 | **`会展地图高保真`** 未拉取或相对路径被破坏 | 确认 LFS；保持该目录与 `index.html` 相对位置不变 |
| 控制台提示 Firebase 初始化失败 | **`firebaseConfig` 错误或项目未启用服务** | 核对 Firestore / FCM 是否启用；两处配置是否一致 |
| Service Worker 注册失败 | 使用 **`file://`** 或非安全上下文 | 改用 **`http://localhost`** 或 **HTTPS**；确认根目录存在 **`firebase-messaging-sw.js`** |
| 地图不出现、无 `AMap` | **Key 未通过前端校验** 或需 **安全密钥** | 检查 **`STARTRACK_AMAP_KEY`**；按高德文档配置 **jscode** 等 |
| 无法获取 FCM token | **通知权限被拒**、或 **SW 未注册** | 在浏览器设置中允许本站通知；查看 Console 报错 |
| Firestore 写入失败 | **安全规则拒绝** | 按 [第 5.4 节](#54-cloud-firestore-安全规则) 调整规则 |
| CDN 样式丢失 | **无外网** 或 **CDN 被拦截** | 检查网络、代理、广告拦截插件 |

---

## 8. 仓库结构（与 GitHub 根目录一致）

```
（仓库根目录）
├── index.html                      # 主应用入口（SPA）
├── firebase-messaging-sw.js        # FCM Service Worker（firebaseConfig 须与 index.html 一致）
├── .gitattributes                  # Git LFS 跟踪规则（以实际内容为准）
├── LICENSE                         # 若存在
├── README.md                       # 本说明
└── 会展地图高保真/                  # 页面引用的媒体资源（相对路径 ./会展地图高保真/...）
    ├── 个人/
    ├── 组合/
    └── …
```

---

## 9. 声明

本项目用于 **课程或原型演示**。**Firebase、高德** 等第三方服务的使用须遵守各自用户协议与配额限制；**密钥泄露、规则误配** 导致的数据与费用风险由使用者自行承担。**请勿**将真实生产密钥推送到公共代码托管平台。
