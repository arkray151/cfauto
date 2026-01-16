---

# 🚀 Cloudflare Worker 多账号智能中控 (V7.0 ECH Support)

这是一个基于 Cloudflare Worker 的高级部署管理工具。它允许你通过一个统一的 Web 面板，同时管理多个 Cloudflare 账号下的多个 Worker 项目（支持 `CMliu-EdgeTunnel`、`Joey-相信光` 以及最新的 `ECH-WebSocket`）。

**V7.0 核心更新：** 新增对 **ECH** 项目的支持，并集成了全球 ProxyIP 选择器，支持代码级自动注入 ProxyIP，无需手动修改代码。
<img width="706" height="419" alt="image" src="https://github.com/user-attachments/assets/6fb09de1-a075-41e0-9cec-2a18f8b6f133" />
<img width="1920" height="892" alt="image" src="https://github.com/user-attachments/assets/2be4ecbd-5704-49c4-a5d3-c87e4b552d66" />

---

## ✨ 核心功能

1. **多账号统一管理**：
* 支持无限添加 Cloudflare 账号（需提供 Account ID 和 API Token）。
* 支持跨账号管理 Worker，每个账号可配置多个不同类型的 Worker。
* **流量监控**：自动获取每个账号的今日流量使用情况，并按剩余流量智能排序。


2. **多项目支持**：
* 🔴 **CMliu (EdgeTunnel)**：支持变量管理、自动更新、UUID 自动轮换。
* 🔵 **Joey (相信光)**：支持自动添加 `window` polyfill 修复，一键部署。
* 🟢 **ECH (WebSocket)**：**[新]** 专为 ECH 项目设计，支持**直接注入 ProxyIP** 到代码中，内置全球/亚洲/欧洲/北美节点列表。


3. **智能部署与更新**：
* **自动检测更新**：对比 GitHub 上游仓库与本地部署版本。
* **一键部署**：从 GitHub 拉取最新纯净代码，注入配置后推送到你的 Worker。
* **配置同步**：支持从已有的 Worker 中读取配置，或将配置同步到其他账号。
* **自动修复**：删除变量时会自动触发代码重部署，防止残留配置导致错误。


4. **自动化 (Cron)**：
* **定时检查更新**：后台自动检查上游更新并推送到所有账号。
* **流量熔断保护**：当账号流量超过设定阈值（如 90%），自动轮换 UUID 并重新部署，防止被滥用。


5. **Web 面板特性**：
* 密码访问保护。
* 响应式布局（支持手机/PC）。
* **布局切换**：支持“上下结构”和“左右分栏”切换，适应不同屏幕。



---

## 🛠️ 部署准备

在部署此中控 Worker 之前，你需要：

1. **一个 Cloudflare 账号**（用于托管此中控脚本）。
2. **一个 KV Namespace**：用于存储账号数据和配置信息。

### 部署步骤

1. 在 Cloudflare Dashboard 创建一个新的 Worker（例如命名为 `manager`）。
2. 将 `worker.js` 的完整代码复制并粘贴到编辑器中，保存。
3. **绑定 KV 命名空间**：
* 进入 Worker 的 **Settings (设置)** -> **Variables (变量)** -> **KV Namespace Bindings**。
* 点击 `Add Binding`。
* **Variable name (变量名)** 必须填写：`CONFIG_KV`。
* **KV Namespace** 选择你创建的 KV。


4. **设置环境变量**：
* 进入 **Settings (设置)** -> **Variables (变量)** -> **Environment Variables**。
* 添加以下变量：



| 变量名 | 必填 | 说明 | 示例 |
| --- | --- | --- | --- |
| `ACCESS_CODE` | ✅ 是 | Web 面板的访问密码（直接在 URL 后加 `?code=密码` 或通过 Cookie 登录） | `MySuperPass123` |
| `GITHUB_TOKEN` | ❌ 否 | (推荐) GitHub PAT Token，用于提高 API 请求限额，防止检查更新频繁报错 | `ghp_xxxxxx` |

---

## ⚙️ Cron 定时任务设置

为了让自动更新和流量监控生效，你需要配置 Cron 触发器。

1. 进入 Worker 的 **Settings (设置)** -> **Triggers (触发器)**。
2. 点击 **Add Cron Trigger**。
3. **推荐配置**：每 30 分钟执行一次。
* Cron 表达式：`*/30 * * * *`


4. 保存。

**Cron 运行逻辑：**

* 脚本会根据你在 Web 面板顶部设置的 **"自动检测"** 开关和 **"时间间隔"** 来决定是否执行实际逻辑。
* 即便是 Cron 每分钟触发一次，如果面板设置的是 30 分钟，脚本也会在未到时间时直接跳过，节省资源。

---

## 📖 使用指南

### 1. 添加被控账号

* 点击面板上的 **"➕ 添加账号"**。
* **Account ID**：在 Cloudflare 首页域名列表右下角获取。
* **API Token**：
* 需在 [My Profile -> API Tokens](https://dash.cloudflare.com/profile/api-tokens) 创建。
* **必须具备的权限**：
1. `Account` - `Cloudflare Pages` - `Edit` (可选，若管理 Pages)
2. `Account` - `Workers Scripts` - `Edit` (必须，用于部署代码)
3. `Account` - `Account Settings` - `Read` (必须，用于读取 ID)
4. `Account` - `Account Analytics` - `Read` (必须，用于统计流量)




* **Worker 名称**：
* 在对应的输入框（红/蓝/绿）填入该账号下**已创建的空 Worker 名称**。
* 支持多个，用逗号分隔（例如：`worker1, worker2`）。



### 2. 管理 ECH 项目 (新功能)

* 在面板右侧（或下方）找到 **🟢 ECH 配置** 卡片。
* **ProxyIP 选择**：
* 使用下拉菜单选择你需要的节点（全球/亚洲/欧洲/北美）。
* 选择后，脚本会自动将其填入 `PROXYIP` 变量框。


* **部署原理**：
* 点击 **"🚀 部署 ECH"** 时，脚本会拉取 GitHub 最新代码。
* 自动查找代码中的 `const CF_FALLBACK_IPS = [...];`。
* 将你选择的 ProxyIP **直接替换** 到代码中（硬编码），然后上传到 Cloudflare。
* *注意：因为是直接修改代码，所以 Cloudflare 后台的环境变量里可能看不到 PROXYIP，这是正常的。*



### 3. 全局设置

* **布局切换**：点击顶部的 `◫ 布局` 按钮，可切换左右分栏模式，方便宽屏操作。
* **熔断阈值**：
* 在顶部设置“熔断”百分比（例如 90）。
* 当某账号流量使用超过 90% 时，Cron 任务会自动为该账号下的 cmliu/joey 项目更换 UUID 并重新部署，防止流量耗尽或被封锁。



---

## ⚠️ 注意事项

1. **KV 绑定至关重要**：如果不绑定名为 `CONFIG_KV` 的 KV 空间，所有数据（账号、配置）都无法保存，网页会报错。
2. **API Token 权限**：如果部署失败（提示 `Authentication` 或 `Permission` 错误），请检查被控账号的 API Token 是否包含了 **Workers Scripts: Edit** 权限。
3. **覆盖风险**：点击部署会**强制覆盖**线上 Worker 的代码。如果你在线上 Worker 手动修改了代码（非环境变量），部署后会被 GitHub 的纯净版本覆盖。
4. **ECH 的 ProxyIP**：ECH 项目不需要频繁更新。如果只是想换 IP，在面板选择 IP 后再次点击部署即可。
5. **GitHub API 限制**：如果不配置 `GITHUB_TOKEN`，频繁刷新页面或检查更新可能会触发 GitHub 的 API 速率限制（每小时 60 次），导致显示“获取失败”。建议在环境变量中配置 Token。

---

## 📅 更新日志

### V7.0

* [新增] **ECH 项目支持**：针对 WebSocket 协议的特定优化。
* [新增] **ProxyIP 选择器**：内置常用高质量 ProxyIP 列表。
* [核心] **代码注入技术**：部署 ECH 时自动替换源码中的常量。
* [优化] 账号编辑界面适配三种项目类型。

### V6.8

* [修复] 删除变量时自动重置代码，解决残留配置导致的 Worker 崩溃问题。
* [优化] 增加“同步配置”时的账号选择功能。
* [UI] 增加布局切换功能。
