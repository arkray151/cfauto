# 🚀 Cloudflare Worker 多项目部署管理器 (Ultimate Edition)

这是一个运行在 Cloudflare Worker 上的**高级中控面板**。它允许你通过一个简单的网页界面，批量管理、监控和更新你的所有 VLESS/VMess 节点。

特别针对 **CMliu (EdgeTunnel)** 和 **Joey (CFNew)** 两个项目进行了深度定制和隔离，支持版本检测和代码自动修复。

## ✨ 核心功能

* **🔄 双模引擎 (项目隔离)**：
* **🔴 CMliu 模式**：深度适配 EdgeTunnel，全量管理 UUID、PROXYIP 等变量。
* **🔵 Joey 模式**：适配 CFNew，**自动注入 `window` 补丁**，彻底解决混淆代码报错问题。


* **📡 账号聚合管理**：
* 只需添加一次 Account ID 和 API Token。
* 支持分别填写 CMliu 和 Joey 的 Worker 名称（逗号分隔），自动分流管理。


* **UP 版本实时检测**：
* 集成 GitHub API，实时对比上游仓库最新 Commit 和本地部署版本。
* 界面直观显示 **“🔴 发现新版本”** 或 **“✅ 已是最新”**。


* **⚡️ API 速率突破**：
* 支持配置 `GITHUB_TOKEN`，将 GitHub API 请求额度提升至 5000次/小时，告别检测失败。


* **☁️ 云端 KV 存储**：
* 所有配置（账号、变量、版本记录）存储在 Cloudflare KV 中，跨设备同步，刷新不丢失。


* **🛠 智能变量补全**：
* 自动检查并补全缺少的默认变量，支持一键刷新 UUID。



---

## 🛠️ 部署教程

### 1. 准备工作

* **Cloudflare**: 获取 `Account ID` 和 `API Token` (必须使用 **Edit Cloudflare Workers** 模板)。
* **GitHub**: 获取 `Personal Access Token` (用于解除 API 速率限制，推荐)。

### 2. 创建 KV 命名空间

1. 登录 Cloudflare 控制台 -> **Workers & Pages** -> **KV**。
2. 点击 **Create a Namespace**。
3. 命名为 `CONFIG_KV` (建议)，点击添加。

### 3. 部署 Worker 代码

1. 新建一个 Worker。
2. 将本项目提供的 `worker.js` (最终完整版) 全量粘贴进去。

### 4. 绑定环境变量 (关键步骤)

进入 Worker 的 **Settings (设置)** -> **Variables (变量)**：

#### A. 绑定 KV (必须)

* 点击 **KV Namespace Bindings** -> **Add binding**。
* **Variable name**: `CONFIG_KV` (必须完全一致)。
* **KV Namespace**: 选择第 2 步创建的 KV。

#### B. 设置 GitHub Token (推荐)

为了防止检查更新时出现“速率限制”报错：

* 点击 **Environment Variables** -> **Add variable**。
* **Variable name**: `GITHUB_TOKEN`。
* **Value**: 填入你的 GitHub PAT (`ghp_xxxx...`)。

#### C. 设置访问密码 (可选)

* **Variable name**: `ACCESS_CODE`。
* **Value**: 设置你的登录密码。

---

## 📖 使用指南

### 1. 添加账号 (聚合模式)

在左侧面板填写账号信息：

* **Account ID** & **API Token**: 填入一次即可。
* **🔴 CMliu Workers**: 输入运行 EdgeTunnel 的 Worker 名称，用逗号隔开 (如: `hk-01, sg-01`)。
* **🔵 Joey Workers**: 输入运行 CFNew 的 Worker 名称，用逗号隔开 (如: `joe-01`)。

### 2. 检查更新 & 部署

1. **切换项目**: 在右上角下拉菜单选择 **CMliu** 或 **Joey**。
* *系统会自动加载该项目对应的变量和 Worker 列表。*
* *系统会自动联网检测 GitHub 上游是否有新版本。*


2. **查看状态**:
* 如果显示红色 **🔴 发现新版本**，建议更新。


3. **配置变量**:
* 在右侧面板修改 UUID 或其他参数 (支持一键刷新)。


4. **一键更新**:
* 点击底部的 **🔄 立即执行更新**。
* 脚本会自动拉取最新代码 -> 注入补丁(如果是Joey) -> 打包变量 -> 推送更新。



---

## ❓ 常见问题

**Q: 为什么右上角一直显示“检测失败”？**
A: 很有可能是 GitHub API 触发了每小时 60 次的速率限制。请务必在 Worker 设置里添加 `GITHUB_TOKEN` 环境变量。

**Q: 为什么 Joey 的代码部署后能运行，但我自己上传就报错？**
A: Joey 的代码经过了混淆，使用了浏览器环境的 `window` 对象。本工具在部署 Joey 项目时，会自动在代码头部插入 `var window = globalThis;`，这是本工具的独家修复功能。

**Q: 我切换到 CMliu 项目，Joey 的变量会丢失吗？**
A: 不会。变量是严格隔离存储的。CMliu 的变量存在 `VARS_cmliu`，Joey 的存在 `VARS_joey`，互不干扰。

**Q: 为什么点击更新后，Worker 变成了 404？**
A: 如果你是第一次添加某个 Worker 名称，Cloudflare 需要 1-2 分钟来完成 DNS 传播和路由注册，请稍等片刻即可访问。

---

## ⚠️ 免责声明

本项目仅供技术研究和学习使用，请勿用于非法用途。使用本工具产生的任何后果由使用者自行承担。
