# 🕒 RenewHelper - 时序·守望 (Service Lifecycle Manager)

![Docker Pulls](https://img.shields.io/docker/pulls/ieax/renewhelper?logo=docker)

**RenewHelper - 时序·守望** 是一款基于 **Cloudflare Workers** **

<div align="center">
  <img src="./assets/mainUI_darkCN_shotv3.png" alt="RenewHelper 界面预览" width="800">
  <img src="./assets/CalendarUI_darkCN_shotv3.png" alt="RenewHelper 界面预览" width="800">
  <img src="./assets/DashboardUI_darkCNv3.png" alt="RenewHelper 界面预览" width="800">
</div>

## ✨ 核心特性

- **⚡️ Serverless 架构**：完全运行在 Cloudflare Workers 上，利用 KV 存储数据，无需购买 VPS，免费额度通常足够个人使用。并同时支持单机Docker方式部署，不依赖任何CDN引入即可独立运行。
- **🛡️ 安全可靠**：
  - JWT 身份验证，支持高强度密钥自动生成。
  - 混合限流策略（内存 + KV），防止暴力破解。
  - 数据仅存储在您私有的 Cloudflare KV 中。
  - 敏感操作（删除、重置）二次确认。
- **🎨 现代化 UI**：
  - Vue 3 + Element Plus 构建的单文件前端。
  - 支持深色/浅色模式切换，并内置平滑的视图切换过渡动画。
  - 支持便捷的列表多选与批量操作（批量删除、启停、分配通知渠道）。

---

## 🚀 部署指南

### 方式一：自动一键部署 (推荐)

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/ieax/renewhelper)

点击授权CF访问您的Git账户-按指引完成后进入CF变量-添加`AUTH_PASSWORD`值`admin`

### 方式二：网页手动部署 (新手推荐)

如果您没有 Github，可以直接在 Cloudflare 网页后台完成部署。

#### 第一步：创建 KV 存储桶

登录 CF **Workers & Pages** -> **KV**
**Create a namespace** `RENEW_KV`

#### 第二步：创建 Worker 服务

**Overview**---**Create application**---**Create Worker**。
**Deploy**成功后点击**Edit code**进入代码编辑器粘贴`worker.js`替换所有内容并**Deploy**

#### 第四步：绑定 KV 数据库 (关键)

**Settings** -> **Variables** --- **KV Namespace Bindings** --- **Add binding**：
**Variable name**: 必须填 `RENEW_KV`
**KV Namespace**: 下拉选择存储桶后**Save and deploy**。

#### 第五步：设置登录密码

**Variables** --- **Environment Variables** --- **Add variable**
**Variable name**: `AUTH_PASSWORD`=`admin`
**Save and deploy**。

#### 第六步：设置定时任务 (Cron)

为了让自动续期功能生效，**须要设置定时触发器。**

触发器**Triggers** --- **Cron Triggers**
**Add Cron Trigger** --- **Cron schedule**: 准确输入 `0,30 * * * *` --- **Add Trigger**

### 方式三：GitHub Actions 部署 (进阶推荐)

此方法适合希望**自动同步更新**且关注**隐私安全**的用户。您无需授权第三方应用，所有密钥均存储在您自己的 GitHub 仓库中。

1.  **Fork 项目**：点击本项目右上角的 **Fork** 按钮，将仓库复制到您的 GitHub 账户。
2.  **准备 Cloudflare 密钥**：
    - **Account ID**：登录 Cloudflare Workers 首页右侧获取。
    - **API Token**：[My Profile](https://dash.cloudflare.com/profile/api-tokens) -> API Tokens -> Create Token -> 选择 **Edit Cloudflare Workers** 模板 -> 生成并复制。
3.  **创建 KV 数据库**：
    - 在 Cloudflare 后台创建一个 KV 命名空间（如 `RENEW_KV`）。
    - 复制该 KV 的 **ID**。

4.  **配置 GitHub Secrets**：
    - 进入您 Fork 的仓库 -> **Settings** -> **Secrets and variables** -> **Actions**。
    - 点击 **New repository secret**，依次添加4个变量：
        - `CF_API_TOKEN`: 填入您的 API Token。
        - `CF_ACCOUNT_ID`: 填入您的 Account ID。
        - `CF_KV_ID`: 填入您刚刚复制的 KV ID (Actions 会自动注入)。        
        - `AUTH_PASSWORD`: 填入您的登录密码(设置后，即使同步代码也不会被覆盖)。

5.  **启用并部署**：
    - 进入 **Actions** 标签页，点击绿色按钮 **I understand my workflows...** 启用。
    - 在左侧选择 **Deploy to Cloudflare Workers**，点击右侧 **Run workflow** 手动触发首次部署。
    - **后续更新**：每当原作者发布新版本，您只需在 GitHub 点击 Sync Fork，Actions 会自动将最新代码（含新功能）部署到您的 Worker，同时保留您的密码设置。

```

### Telegram 代理服务部署 (可选)

> ⚠️ **仅适用于中国大陆用户**：由于网络原因，国内服务器可能无法直接连接 Telegram API。您可以部署本项目提供的轻量级反代，同时兼容Worker/Pages/Snippets，建议部署方式：Snippets（无次数限制速度快） > Pages（无次数限制） > Worker（有次数限制）。

1.  **准备文件**：复制本项目 `renewhelper/telegram_proxy/_worker.js` 的代码。
2.  **创建 Worker/Pages/Snippets**：
    *   在 Cloudflare 后台创建一个新的 Worker/Pages/Snippets（例如命名为 `tg-proxy`）。
    *   粘贴代码并部署。
3.  **配置白名单 (变量名: `TG_ALLOW_TOKENS`)**：
    *   在 Worker/Pages 设置 -> 变量中，添加 `TG_ALLOW_TOKENS`(Pages需要重新部署一遍生效)。
    *   值填写您的 Bot Token（如果有多个用逗号分隔）。
    *   *如果不配置环境变量，需手动修改代码中的 `WHITELIST_TOKENS` 常量。*
4.  **使用**：
    *   您的代理地址为：`https://tg-proxy.您的子域名.workers.dev` 或您的自定义域名。
    *   此服务可作为 Telegram API 的透明代理，支持 `POST /bot<Token>/<Method>` 格式的请求。
    *   *注：此脚本主要用于解决 Docker 部署环境下或特定网络环境下无法访问 Telegram API 的问题。*

### 🎉 部署完成！

---

## ⚙️ 设置与配置

部署成功后，访问您的 Worker 域名或您添加的自定义域名（如 `https://renewhelper.your-name.workers.dev`或 `https://renewhelper.your-domain.com`）。

1.  **首次登录**：使用您设置的 `AUTH_PASSWORD`（默认 `admin`）解锁。
2.  **系统设置**：点击控制台右上角的 **系统设置 (SETTINGS)** 按钮。
    - **时区设置**：非常重要！请选择您所在的时区（如 `Asia/Shanghai`），这决定了提醒和日历的准确性。
    - **通知总开关**：开启后可配置具体的推送渠道。

<div align="center">
  <img src="./assets/configUI_darkCN_shotv3.png" alt="RenewHelper 界面预览" width="800">
</div>

### 📢 推送渠道配置说明

在“系统设置” -> “通知配置”区域，点击 **添加渠道** 按钮，选择类型并填写参数。系统支持同时配置**无限个**推送渠道。配置完成后支持**发送测试**以验证连通性，并可通过列表上方的**全选/反选**及**批量操作栏**进行启用、禁用、删除或**一键分配**给指定服务。

| 渠道              | 参数说明                                                           | 获取/配置方法                                                                                                                                                                                                                               |
| :---------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Telegram**      | **Token**: 机器人令牌<br>**Chat ID**: 您的用户 ID 或群组 ID<br>**Server**: (可选) 自定义 API 地址 | 1. 找 [@BotFather](https://t.me/BotFather) 创建机器人获取 Token。找`@userinfobot`获取UserID。<br>2. 或浏览器访问 `https://api.telegram.org/bot<YourToken>/getUpdates` <br>3. 添加您的机器人为好友，并发送任意消息给机器人。<br>4. 在刚才打开的 URL 页面刷新获取 Chat ID。<br>5. **Server**: 默认为 `https://api.telegram.org`。如需使用反代，请填入完整地址（如您的 [Telegram 反代服务](#telegram-代理服务部署-可选) URL`https://tg-proxy.your-name.pages.dev`）。 |
| **Bark** (iOS)    | **Server**: 服务器地址<br>**Device Key**: 设备密钥                 | 1. App Store 下载 Bark 应用。<br>2. 复制 App 内显示的服务器地址和 Key。                                                                                                                                                                     |
| **PushPlus**      | **Token**: 用户令牌                                                | 1. 访问 [PushPlus 官网](https://www.pushplus.plus/)。<br>2. 微信扫码登录获取 Token。                                                                                                                                                        |
| **NotifyX**       | **API Key**: 密钥                                                  | 1. 访问 [NotifyX 官网](https://www.notifyx.cn/)。 <br>2. 微信扫码登录获取 API Key。                                                                                                                                                         |
| **Resend** (邮件) | **API Key**: Resend 密钥<br>**From**: 发件地址<br>**To**: 收件地址 | 1. 注册 [Resend](https://resend.com/)。<br>2. 绑定域名并获取 API Key。<br>3. `From` 必须是您验证过的域名邮箱（如 `alert@yourdomain.com`）。若您没有域名邮箱，可以使用`onboarding@resend.dev`，发送至您注册 resend 账号的邮箱。              |
| **Lark** (飞书)   | **Token**: Webhook 地址中的 UUID<br>**Secret**: (可选) 安全签名密钥 | 1. 在飞书群设置 -> 机器人 -> 添加机器人 -> 自定义机器人。<br>2. 复制 Webhook 地址结尾的 UUID（例如：`xxxxxxxx-xxxx-...`）填入 Token 中。<br>3. 选填加签密钥。                                         |
| **WeCom** (企微)  | **Key**: Webhook 地址中的 key 参数                              | 1. 在企业微信群设置 -> 添加群机器人。<br>2. 获取 Webhook 地址，并仅提取 `key=` 后面的参数名部分（例如：`693...`）。                                                           |
| **Gotify**        | **Server**: 服务器地址<br>**Token**: 应用 Token                    | 自建 Gotify 服务器，创建一个 Application 获取 Token。                                                                                                                                                                                                       |
| **Ntfy**          | **Server**: 服务器 (默认 ntfy.sh)<br>**Topic**: 主题<br>**Token**: 令牌 | 1. **Server**: 若自建则填自建地址，否则留空默认为 `https://ntfy.sh`。<br>2. **Topic**: 您订阅的主题名称。<br>3. **Token**: (可选) 如果主题受保护，需填写 Access Token，否则留空。                                                                           |
| **Server酱3**     | **UID**: 用户ID<br>**SendKey**: 发送密钥                       | 1. 登录 [Server酱3](https://sc3.ftqq.com/)。<br>2. 获取 UID 和 SendKey。                                                                                                                                                                          |
| **钉钉** (DingTalk)| **Token**: access_token<br>**Secret**: (可选) 加签密钥 | 1. 钉钉群设置 -> 智能群助手 -> 添加机器人 -> 自定义。<br>2. 安全设置推荐勾选 **加签** (Secret) 或 **自定义关键词** 。<br>3. 复制 Webhook 地址中的 `access_token` 和加签的 `SEC...` (即 Secret)。若使用自定义关键词记得放行`Renew`，否则无法收到通知。                                                           |
| **Webhook**       | **URL**: POST 地址                                                 | 适用于自定义开发。系统会向该 URL 发送 POST 请求：`{ "title": "...", "content": "..." }`。[WEBHOOK 配置教程](./webhook_guide_zh.md)                                                                                                                                                   |

---

## 🛠 使用方法

### 添加服务

<div align="center">
  <img src="./assets/AddUI_darkCN_shotv3.png" alt="RenewHelper 界面预览" width="600">
</div>

- **名称**：服务的名称（如 "Netflix 4K"，"Google Voice - 8888"）。
- **标签**：用于分类（如 `Media`, `Server`, `Domain`, `PhoneNumber`），支持多选。
- **模式**：
  - 📅 **循环订阅(Cycle)**：每隔固定周期（如 1 个月/1 个自然年/1 个农历年）到期的各种事项，如月付会员订阅、年付 VPS 续费等。
  - ⏳ **到期重置(Reset)**：到期后需手动或自动处理，有效期随之展期的各种事项，如 eSIM 动账延长 180 天有效期、签到增加服务时长等。
  - 🔁 **固定重复(Repeat)**：基于强大的 RRULE 规则引擎打造。适用于“每月的第二个星期五”、“每两个星期的周三和周日”这类按自然习惯定期的事件。
- **农历开关**：开启后，周期将按农历计算（适合农历生日、事物提醒）。
- **提前提醒**：可自由选择需要提前多少天收到通知，支持提醒时间**多选**（例如同时勾选“8:00”和“18:00”接收推送）。
- **自动化策略**：
  - **自动续期**：到期后自动将下次到期日顺延一个周期。
  - **自动禁用**：到期超过指定天数未处理，自动标记为禁用。


### ICS 日历订阅 & 日历视图

在主界面不仅可以点击右上角切换至**日历直观视图**（不仅直观展示当月的续费事件，还可以点击顶部开关，显示未来推测的待付项目点）。
在“系统设置”选项中更可获取**日历订阅**提取网址：

1.  复制订阅链接。
2.  **iOS**: 设置 -\> 邮件 -\> 账户 -\> 添加账户 -\> 其他 -\> 添加已订阅的日历。
3.  **Google Calendar**: 添加日历 -\> 来自 URL 或从订阅链接下载后文件导入。
4.  **Outlook**: 文件 -\> 添加日历 -\> 从 WEB 订阅或从订阅链接下载后文件导入。
5.  **手机自带日程 APP**: 在 CalDAV 服务商添加订阅链接，手机添加 CalDAV 订阅信息进行日程同步或从订阅链接下载后，在日程软件中导入。
6.  **注意**: 链接包含安全 Token，请勿泄露。如泄露可点击“重置令牌”。

### 💰 资金流向 (Billing Stats)

### 📜 历史账单 (History Records)

1.  **手动续费 (Renew)**：适合当期续费。系统会自动根据上一周期的结束时间推算下一次的起止时间填入。
2.  **补录历史 (Add History)**：适合补录过去的账单。点击“切换为补录模式”，您可以手动添加任意时间段的账单记录。
3.  **编辑记录**：点击右侧的“历史记录”按钮，可以查看并修正每一笔过往的账单详情（价格、日期、备注）。



---

## ⚠️ 注意事项

1.  **数据安全**：所有数据存储在您的 Cloudflare KV 中, 建议定期手动导出 JSON 备份。
2.  **免费版限制**：（Docker方式不受此限制）
    - Cloudflare Workers 免费版每天限制 100,000 次请求。
    - KV 每天写入限制 1,000 次。

---


**爱发电** - 国内用户

[![爱发电](https://img.shields.io/badge/爱发电-Afdian-946ce6?style=for-the-badge&logo=github&logoColor=white)](https://afdian.com/a/lostfree)

---

**License**: MIT
Copyright (c) 2025-2026 LOSTFREE
