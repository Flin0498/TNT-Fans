# StarTrack·星途（MICE1901）

[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)](#)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)](#)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](#)
[![Firebase](https://img.shields.io/badge/Firebase-Firestore%20%7C%20FCM-FFCA28?style=flat-square&logo=firebase&logoColor=black)](#)
[![Git LFS](https://img.shields.io/badge/Git-LFS-F05133?style=flat-square&logo=gitlfs&logoColor=white)](#)

嗨，感谢点开这份说明。下面会尽量用**好懂、不吓人**的方式，把怎么跑起来、哪里要配 Key、哪里其实不用折腾，都写清楚——**该保留的技术细节和链接一个都不会少**。

---

## 1. 项目简介

**StarTrack·星途** 是一款面向演唱会与大型会展（MICE）场景的 **移动端 Web 单页应用（SPA）** 原型，想给大家演示的是一整条动线：从 **活动发现**，到 **行程与路线**，再到 **现场地图与安全**，还有 **粉丝社区与内容**、**出行协作**，最后落到 **个人中心**。打开 **`index.html`** 就能从入口玩起来——后文说的 **「仓库根目录」**，指的就是你克隆下来的**项目根路径**（和 `index.html` 同级的那一层）。

界面好看主要靠 **Tailwind CSS**，小图标用 **Font Awesome**，都是通过 **CDN** 拉的，所以这台机器要 **能上公网**，第一次打开时样式才齐。

**重要限制（建议认真扫一眼，以后少踩坑）**：

*   **本机不用装「Firebase / 高德软件」才能看网页**：Firebase、高德的 **Web SDK 是浏览器打开页面时从 CDN 拉的**（如 `www.gstatic.com`、`webapi.amap.com`），**不装** Firebase CLI、不装高德 PC 客户端也能浏览；CLI 多用于部署/后端，和本仓库**静态前端**无必然关系。地图要正常画，需要你在 **`index.html`** 里填有效的高德 Web Key（见 [第 5.2 节](#52-高德地图开放平台与-key)）。
*   **Git LFS**：仓库里不少大文件走 **Git LFS**。要是没装好、没 `pull` 到位，你打开的 `index.html` 可能只有几行「指针文」，**浏览器里就像没内容**。怎么救回来，看 [第 4.2 节](#42-git-lfs-安装与拉取)。
*   **Firebase 与高德 Key**：`firebaseConfig`、`STARTRACK_AMAP_KEY` 都是写在文件里的**字符串**，不是系统里要装的驱动。**占位时**：**Firestore / FCM 等云端能力不可用**，控制台可能报错；**多数页面切换、`localStorage`、纯前端界面**仍可用来走查、录屏。真要上云、要真地图再换成自己的配置，跟 [第 5 节](#5-配置指南) 做即可。
*   **「开始导航」类按钮**：部分逻辑会跳 **`amapuri://`**，尝试唤起**手机上的高德地图 App**；PC 上没对应客户端时，常见「打不开链接」或跳转商店——与 Firebase 是否成功无关；页内 **嵌入式地图**（`AMap.Map`）能否显示，看的是 **高德 Web Key + JS 是否加载成功**（两条线）。
*   **`file://` 打开方式**：如果你习惯**双击** `index.html` 用本地文件协议打开，代码里会 **主动跳过 Service Worker 注册**，所以 **FCM 推送没法按完整链路测**；地图、Firebase 能不能用，还是看你有没有配好。和 HTTP 的差别写在 [第 4.4 节](#44-本地文件协议与-http-的区别)。
*   **关于启动方式**：这份 README **不会**教你用仓库里可能存在的 **PowerShell 一键脚本**；想起本地服务，用大家更熟的 **Node、`python -m http.server`、VS Code Live Server、Caddy、Nginx** 之类就好。
*   **密钥安全**：带真实 `apiKey`、高德 Key 的改动，**别顺手 push 到公开仓库**，拜托了。

---

## 2. 核心功能

*   **账号与会话**
    *   登录、注册、实名认证——该有的页面流程都在。
    *   会用 **`localStorage`**（例如键 `startrack.session.v1`）记一点本地会话（比如手机号），再配合 **`window.pageHistory`** 做「能往回走」的栈式导航，尽量避免点着点着没路可退。
    *   本地没会话时，一般会从 **登录栈** 进场；有会话的话，会回到 **首页栈**。

*   **首页与活动发现**
    *   活动卡片、演唱会专题、搜索、收藏入口都集中在这儿。
    *   从首页能钻进 **行程、应援一日游、当地探索、打卡/攻略、粉丝社区、已生成路线管理** 等模块。

*   **行程、路线与活动详情**
    *   **行程页、自定义路线、路线结果、路线详情** 之间会互相联动。
    *   **活动详情、接驳/班车详情、专题演唱会页** 负责展示和跳转。
    *   带地图的区块要 **高德 JS API** + 有效 Key，具体怎么申请写在 [第 5.2 节](#52-高德地图开放平台与-key)。

*   **现场、直播与安全**
    *   **直播相关页、现场地图、直播推送设置、陪伴设置** 等，帮你把「人在现场」那条线串起来。
    *   **安全热力、安全中心、告警详情、举报表单** 等，是安全向的信息架构。
    *   有些演示/实时数据在设计上会跟 **Firestore** 集合打交道（要开服务、写规则，别忘啦）。

*   **社区与 UGC**
    *   **粉丝社区、发布攻略、攻略列表与详情、观演评价生成与预览** 等，偏内容和互动演示。

*   **出行与协作（原型级）**
    *   **拼车、拼房、群组聊天、群聊房间** 更多是页面和跳转的**原型占位**，用来展示信息架构；真要实时聊天还得另上后端，那就不在这份静态原型包里啦。

*   **个人中心与系统设置**
    *   **个人资料、收藏、物品清单与编辑、隐私设置、通用设置、帮助与反馈、关于** 等，该有入口的都有。

*   **工程与体验增强**
    *   **图片懒加载**：优先 **`IntersectionObserver`**，老浏览器就老实一点直接塞 `src`。
    *   **卡片展开/收起**：`toggleCard` 一类函数管显隐和图标转一转。
    *   **全站搜索**：自带繁简归一化、模糊匹配和索引，各处的搜索框都能蹭这份逻辑。

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

*   **操作系统**：Windows、macOS、Linux 都行，按你习惯来。
*   **浏览器**：更推荐 **Chrome** 或 **Edge（Chromium）**，看 **Console**、折腾 **Service Worker / Application** 面板会顺手很多。
*   **Git**：克隆、更新仓库用得上，[官方下载](https://git-scm.com/downloads)。
*   **Git LFS**：这个仓库**真的用到了** **Git Large File Storage**，[官网在这](https://git-lfs.com/)。
*   **可选 Node.js**：想跑 `npx serve` 的话装一个。
*   **可选 Python 3**：想跑 `python -m http.server` 的话装一个。
*   **网络**：第一次打开页面时，**Tailwind / Font Awesome / Firebase / 高德 CDN** 等外链都要能访问到。

### 4.2 Git LFS 安装与拉取

**为啥非提这一嘴不可？**  
如果 `.gitattributes` 把 `index.html`、`firebase-messaging-sw.js` 或 `会展地图高保真/` 之类划进了 LFS，那 Git 里存的就先是 **一小段指针文**；只有你执行了 LFS 拉取，才会换成 **真正的文件内容**。跳过这步的话，常见惨状包括：`index.html` 打开几乎空白、视频播不了、Service Worker 脚本怪怪的……所以别跳过。

**安装 Git LFS（按你的系统选一个）**

*   **Windows**：去 Git LFS 官网下安装包，装好以后**重开一下终端**。
*   **macOS**（有 Homebrew 的话）：`brew install git-lfs`
*   **Linux（Debian/Ubuntu 举例）**：`sudo apt install git-lfs`

**全局启用（每台电脑做一次就够）**

```bash
git lfs install
```

**把仓库克隆下来**

```bash
git clone <你的仓库 HTTPS 或 SSH 地址>
cd <仓库目录名>
```

**把大文件内容真正拉下来**

```bash
git lfs pull
```

**怎么知道成了没？**

*   用编辑器打开 **`index.html`**：如果能看到**大段** HTML 和脚本、文件明显不止几 KB，多半就稳了。
*   要是文件开头是 `version https://git-lfs.github.com/spec/v1` 这种，说明还是 **LFS 指针**——再检查下 LFS 装没装、有没有在仓库目录执行 `git lfs pull`，或者问问维护者远程 LFS 是不是也正常。

### 4.3 在仓库根目录启动本地 HTTP 服务

**为啥推荐 HTTP？**  
在 **`http://localhost`** 或 **HTTPS** 下，浏览器才算进了 **安全上下文**，项目里的 **`navigator.serviceWorker.register('firebase-messaging-sw.js')`** 才有机会成功；纯 **`file://`** 的话代码会直接**跳过注册**，推送那条线就测不完整啦。

**小提醒**：终端里的当前目录要是 **仓库根目录**——一眼能看到 `index.html` 和 `firebase-messaging-sw.js` 的那种。

**方式一：Node（个人最常用）**

```bash
npx --yes serve .
```

终端会吐一个类似 `http://localhost:3000` 的地址，浏览器打开就行（端口以终端说的为准）。

**方式二：Python 3**

```bash
python -m http.server 8080
```

然后浏览器开 `http://localhost:8080/`（端口和你命令里写的一致即可）。

**方式三：其他工具**  
**VS Code Live Server**、**Caddy**、**Nginx**……只要能把 **仓库根目录** 当网站根目录静态托管，并且走 **localhost 或 HTTPS**，原则都一样：**同一个源**下要能访问到 **`index.html`** 和 **`firebase-messaging-sw.js`**。

### 4.4 本地文件协议与 HTTP 的区别

*   **只双击打开 `index.html`（`file://`）**  
    *   **好处**：零命令行，最快看到长什么样。  
    *   **代价**：控制台会告诉你——**不会注册 Service Worker**，**FCM 没法按设计完整玩**；有些浏览器对本地文件还更挑剔。  
    *   **Firebase**：还是会尝试初始化，但网络、配置不对的话，控制台一样会唠叨。

*   **走本地 HTTP**  
    *   **好处**：更像真实上线环境，**Service Worker、通知权限、FCM token** 都能认真测。  
    *   **代价**：你得有 Node / Python 或别的静态服务之一。

### 4.5 相对路径与资源目录

*   视频等媒体走的是 **`./会展地图高保真/...`** 这种 **相对路径**，所以 **别手滑** 把 `index.html` 和 `会展地图高保真` 文件夹的相对位置挪乱，不然容易 **404**。
*   如果你克隆下来就是「根目录即项目」那种结构，**路径不用改**；记得把 **LFS 资源拉全** 就好。

---

## 5. 配置指南

> **什么时候需要啃这一章？** 当你想要 **Firestore 真入库**、**FCM 真推送**，或者让 **高德 Web 地图**在页面里稳稳跑起来的时候，再按下面步骤来。**只做静态演示、录屏交作业**的话，通常可以整章跳过；占位 Firebase 时控制台可能红字，属正常现象（见上文 **「重要限制」** 里 Firebase 那条）。

### 5.1 Firebase 控制台与 Web 应用

**5.1.1 创建项目**  
打开 [Firebase Console](https://console.firebase.google.com/)，登录后点「添加项目」，跟着向导走（演示用可以关掉 Google Analytics，看个人喜好）。

**5.1.2 注册 Web 应用**  
在项目概览里点 **「</>」** 添加 **Web** 应用，把向导给你的 **`firebaseConfig`** 字段记好：`apiKey`、`authDomain`、`projectId`、`storageBucket`、`messagingSenderId`、`appId`。

**5.1.3 启用 Cloud Firestore**  
控制台 **「构建」→「Firestore Database」** 里建库。课设/本地玩可以先用 **测试模式**（注意控制台写的 **规则过期时间**）；要长期挂网上给人看，记得收紧规则（[第 5.4 节](#54-cloud-firestore-安全规则) 有讲）。

**5.1.4 启用 Cloud Messaging（FCM）**  
去 **「构建」→「Cloud Messaging」** 按提示开。若界面让你配 **Web 推送证书 / 密钥对** 之类，最好照官方文档做完，否则有些环境下 **`messaging.getToken`** 会闹脾气。

### 5.2 高德地图开放平台与 Key

**5.2.1 申请 Key**  
打开 [高德开放平台](https://lbs.amap.com/)，注册登录后，在 **「应用管理」→「我的应用」** 里建应用、加 **Key**；服务平台记得选 **「Web 端（JS API）」**，才跟项目里的 **JS API 2.0** 对得上。

**5.2.2 安全密钥（如控制台要求）**  
如果你的 Key 开了 **安全密钥（jscode）** 之类，记得按 [高德 JS API 2.0 加载与安全说明](https://lbs.amap.com/api/javascript-api-v2/guide/abc/load) 配（有的场景要在加载地图脚本前设 **`window._AMapSecurityConfig`**）。**当前仓库默认只在脚本 URL 里带 `key`**；要是鉴权报错，先对照文档补配置，**别急着乱改相对路径**。

**5.2.3 写入 `index.html`**  
在文件前面找到这一段，把引号里的内容换成你的 **Web 端 Key**（别留中文占位、别夹空格）：

```javascript
window.STARTRACK_AMAP_KEY = '您的高德地图AK密钥';
```

**5.2.4 内置校验逻辑（无需改路径）**  
Key 要是空的、太短、含中文、或者还带着「请替换」「密钥」这种占位词，项目会 **干脆不请求** 高德脚本，免得控制台被 **Error key** 淹没。配对了再刷新，在需要地图的页面里，`typeof AMap !== 'undefined'` 那套逻辑就有机会跑起来。

**5.2.5 正式上线**  
别忘了在开放平台给 Key 加 **HTTP Referer** 等限制，省得被别人盗用你的额度。

### 5.3 将 Firebase 配置写入两处代码（必须一致）

咱们项目里 **有两个入口** 都要吃同一份 Firebase 配置，**字段得一模一样**：

| 文件 | 说明 |
| :--- | :--- |
| **`index.html`** | 用 **compat** 脚本初始化 `firebase-app`、`firestore`、`messaging` |
| **`firebase-messaging-sw.js`** | Service Worker 里用 **`importScripts`** 拉 **模块化** 的 `firebase-app.js` 与 `firebase-messaging.js`（版本 **9.23.0**），再 **`firebase.initializeApp(firebaseConfig)`** |

**照着做就行**

1. 在 **`index.html`** 里搜 **`const firebaseConfig = {`**，用控制台那份配置 **整块替换** 占位。  
2. 打开 **`firebase-messaging-sw.js`**，把顶上的 **`firebaseConfig`** **同样改一份**（复制粘贴最省心，少手抖）。  
3. 存盘，用 **localhost 或 HTTPS** 打开站点，看 **Console**：正常的话会有 **Firebase 初始化成功** 一类日志；报错就检查 JSON（英文引号、逗号、别混进注释）。

**再啰嗦一遍**：真配置别往**公开**远程仓库里推；私有分支、本地忽略、或以后改成构建时注入，都随你，但得自己多写点工程活。

### 5.4 Cloud Firestore 安全规则

代码里确实有往 Firestore **写** 东西的逻辑（比如 **`firebase.firestore.FieldValue.serverTimestamp()`**）。规则要是全拒，浏览器里就会看到 **Permission denied** 之类的报错，别慌，那是规则在说话。

*   **开发期**：可以先用控制台给的测试规则，但盯紧 **过期时间**。  
*   **演示/生产**：去 **Firestore → 规则** 里按集合写细一点的 **`allow read, write`**，别长期 **`if true`** 全开给全世界。

具体集合名，自己在 **`index.html`** 里搜 **`db.collection`** 对一下；改完规则记得点 **发布**。

### 5.5 FCM 与 Service Worker 行为摘要

*   注册写的是 **`navigator.serviceWorker.register('firebase-messaging-sw.js')`**，所以默认 **SW 和 `index.html` 躺在同一层根路径**。要是你以后部署到 **子路径**（比如 `https://example.com/app/`），注册 URL 和部署方式要一起改——那就属于进阶话题啦。  
*   **`firebase-messaging-sw.js`** 里通知用到了 **`icon.png`**（相对 SW 自己的地址解析）。根目录没有这个文件的话，通知也许还能弹，但图标可能丑或裂；可以自己丢一个 **`icon.png`** 进去。  
*   页面一加载就会问你要 **通知权限**；用户点了拒绝，**FCM token** 拿不到，这是正常现象，不是 bug。

---

## 6. 使用说明（文字）

下面按**比较自然的上手顺序**讲界面逻辑；**没有配图**，你可以一边开浏览器一边对照。

### 6.1 首次进入与会话恢复

*   一进来会先瞄一眼 **`localStorage`** 里有没有会话；没有的话，**页面历史栈** 会从 **登录** 相关状态起步。  
*   登录流程走完后，一般就能进 **首页**；底部或页面里的按钮会调 **`showPage('…')`** 切到对应区块。

### 6.2 登录、注册与实名认证

*   **登录、注册、实名认证** 各占一块，用 **`showPage`** 切换显示。  
*   密码规则、表单校验以**页面代码**为准（和课程文档不一致时，信代码）。

### 6.3 首页与搜索、收藏

*   **首页** 把活动入口、专题卡片、快捷操作拢在一起。  
*   **搜索** 会跳到搜索相关页面，复用全站搜索那套索引。  
*   **收藏** 进收藏页；数据最后进不进 Firestore，取决于你有没有把云端和规则配好。

### 6.4 行程、路线与活动详情

*   从首页或 **个人中心快捷入口** 能摸到 **行程、应援一日游、当地探索、打卡/攻略、社区、已生成路线** 等。  
*   **行程页** 可以挂演唱会/活动上下文；**地图组件** 吃不吃得到高德，看脚本有没有加载成功。  
*   **活动详情、接驳详情、安全热力** 等子页，靠 **`showPage`** + **`pageHistory`** 把「返回」理顺。

### 6.5 现场、直播与推送相关设置

*   **直播页、现场地图、直播推送设置、陪伴设置** 用来摆现场动线和通知偏好。  
*   在 **HTTP + Firebase 配好 + 用户肯开通知** 的前提下，Console 里可能看到 **FCM token**；也可以去 Firebase 控制台试着发测试消息（界面会随官方改版，这里就不假装截图教程啦）。

### 6.6 社区、攻略与评价

*   **粉丝社区、发攻略、看攻略详情** 走 UGC 那一套演示。  
*   **观演评价生成与预览** 给你看「从填到预览」的步骤感。

### 6.7 拼车、拼房与群聊（原型）

*   **拼车、拼房、群聊、房间页** 主要是**原型级**页面和按钮，展示信息架构；真要实时聊，还得另起后端，这份静态包里就不展开了。

### 6.8 个人中心与设置

*   **个人资料** 里头像、昵称之类，以页面实现为准。  
*   **设置** 里能摸到隐私、推送、关于、帮助反馈等。  
*   **返回**：没带单独 `id` 的 **`.back-button`** 会靠 **`window.pageHistory`** 统一帮你退一步。

---

## 7. 常见问题与排查

| 现象 | 可能原因 | 建议处理 |
| :--- | :--- | :--- |
| 克隆后 `index.html` 几乎为空 | **没拉 Git LFS** | 装好 LFS → `git lfs install` → 进仓库 `git lfs pull` |
| 视频黑屏或无法播放 | **`会展地图高保真`** 没拉全，或相对路径被挪坏了 | 确认 LFS；目录别和 `index.html` 乱相对 |
| 控制台 Firebase 初始化失败 | **`firebaseConfig` 不对** 或控制台里服务没开 | 看 Firestore / FCM 开没开；两处配置是不是复制粘贴同一份 |
| Service Worker 注册失败 | 还在用 **`file://`**，或不是安全上下文 | 换 **`http://localhost`** 或 **HTTPS**；确认根目录有 **`firebase-messaging-sw.js`** |
| 地图没出来、没有 `AMap` | **Key 没通过前端校验**，或还要 **安全密钥** | 看 **`STARTRACK_AMAP_KEY`**；按高德文档补 **jscode** 等 |
| 拿不到 FCM token | **通知被拒** 或 **SW 没挂上** | 浏览器设置里允许通知；再看 Console |
| Firestore 写入失败 | **规则把你挡在门外** | 按 [第 5.4 节](#54-cloud-firestore-安全规则) 调规则 |
| 样式全没了 | **上不了网** 或 **CDN 被挡** | 检查网络、代理、广告拦截插件 |
| 点「开始导航」怪链接或没反应 | 走的是 **`amapuri://` 唤起高德 App**；与嵌入式 Web 地图（`AMap`）是否加载成功是两条线 | 手机上装个高德试最稳；PC 没客户端时很常见 |

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

本项目用于 **课程或原型演示**。**Firebase、高德** 这些第三方服务，怎么用、配额多少，都得遵守人家自己的用户协议；**Key 泄露、规则写飘了** 带来的数据和账单问题，得自己扛。**千万别**把真·生产密钥往公共代码托管平台上随手一推。

---

如果读到这里还有迷糊的地方，多翻翻 **第 7 节** 的表格，再回头扫一眼 **第 1 节「重要限制」**——大部分「我是不是少装了啥」的焦虑，都能对上号。祝你演示顺利。
