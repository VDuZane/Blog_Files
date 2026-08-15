---
title: 邮件域名配置记录（发信 + 收验证码）
slug: email-domains-setup
category: 个人操作记录
category_slug: personal_log
tags: ["邮件", "DNS", "阿里云", "ImprovMX", "运维"]
visibility: admin
status: publish
source: typecho
created: 2026-08-15
---

# 邮件域名配置记录（发信 + 收验证码）

**记录日期：2026-08-15**

> 个人运维记录。  
> 用途：很久以后可根据本文回忆「三个子域各自干什么、DNS 怎么配、博客怎么接、日常要不要管」。

---

## 1. 总览（你实际做成的架构）

| 子域名 | 用途 | 平台 | 状态（记录时） |
|--------|------|------|----------------|
| **notice.liubojian.com** | **发信**（系统验证码 / 通知） | 阿里云 **邮件推送 DirectMail** | 已验证；已接到本地 Typecho 博客并测通发码 |
| **mail.liubojian.com** | **发信**（备用 / 同类发信域） | 阿里云 **邮件推送** | 已验证（与 notice 同类，推送发信域） |
| **inbox.liubojian.com** | **收信转发**（多前缀收验证码） | **ImprovMX** → 转发到主 **QQ 邮箱** | DNS Active；已用该后缀注册 Claude 成功（无需手机号） |

### 心智图

```text
【发信】博客 / SMTP
  notice@… 或 mail@…  （阿里云邮件推送）
       │
       ▼
  用户的 QQ / 163 / Gmail 等

【收信】你在其它网站注册时填写
  任意前缀@inbox.liubojian.com
       │  MX → ImprovMX
       ▼
  自动转发到你的主 QQ 邮箱
```

**原则：发信用推送域，收信用 inbox 转发域，不要混用同一子域的 MX。**

---

## 2. 背景与结论（和讨论相关的要点）

### 2.1 邮件推送 ≠ 企业邮箱

- 阿里云 **邮件推送**：适合 **向外发**（验证码、通知），创建的是「发信地址」，**不是**完整网页收件箱。
- **企业邮箱**：可登录、多账号互发互收，费用较高；个人「多地址收验证码」不必买。
- 需求「多个邮箱名接收别的网站验证码」→ 用 **域名邮箱转发**（ImprovMX）即可，不必自建邮局。

### 2.2 为何不建议自建转发（云 / 个人主机）

- 阿里云 / 家用宽带常限制 **25 端口**，投递与运维成本高。
- 邮件要求 **MX 指向的主机在对方发信时在线**；「用时开机」仍麻烦且不如现成服务稳。
- ImprovMX 免费层已能满足「通配转发到 QQ」；自建性价比低。

### 2.3 ImprovMX 为何能免费

- Freemium：基础转发免费，SMTP 外发 / 更多额度等走付费。
- 日常 **只收验证码不需要付费**；配通后自动转发，不必每次登录操作。

---

## 3. 操作记录：阿里云邮件推送（notice / mail）

### 3.1 做了什么

1. 在阿里云 **邮件推送** 中创建发信域名：
   - `notice.liubojian.com`
   - `mail.liubojian.com`
2. 按控制台「配置」要求，在 **阿里云域名解析**（`liubojian.com`）为这两个子域添加验证所需记录（所有权 TXT、SPF、DKIM、DMARC、MX 等，**以当时控制台值为准**）。
3. 验证通过（记录：相关验证项已通过）。
4. 创建 **发信地址**（例如 `某账号@notice.liubojian.com`），设置 **SMTP 密码**。
5. 将 notice 相关 SMTP 配置写入 **本地 Typecho 博客** 的 PasswordReset（或同类）插件，并完成 **发送验证码测试**。

### 3.2 博客侧（Typecho / 05-typecho）要点

- 工程路径示例：`vdu_blogs/05-typecho`（Docker 运行 Typecho）。
- 插件：**PasswordReset**（邮箱验证码注册 / 重置）。
- 典型配置项：
  - SMTP 主机：如 `smtpdm.aliyun.com`（以邮件推送控制台地域说明为准）
  - 端口：常见 `465` + SSL
  - 用户名 / 发件人：完整发信地址（`…@notice.liubojian.com`）
  - 密码：该发信地址的 SMTP 密码
- **站点地址**（siteUrl）须与浏览器访问地址一致（局域网曾用 `http://192.168.x.x:8085/`），否则会出现「无排版 / CSS 丢失」（与邮件无关，但同属本站运维注意点）。

### 3.3 若曾误把 inbox 也建成「发信域」

- `inbox` 若只在 **邮件推送** 里验证，则 **只能发、不能当收件箱用**。
- 收码应把 `inbox` 的 **MX 交给 ImprovMX**；之后推送里的 `inbox` 发信域验证会失败，**建议在邮件推送中删除 `inbox` 发信域名**，避免混淆。
- **不要删** 仍在使用的 `notice` / `mail`。

---

## 4. 操作记录：ImprovMX 收信（inbox）

### 4.1 做了什么

1. 在 [ImprovMX](https://improvmx.com/) **注册账号**（注册邮箱须用 **已有真实邮箱**，如 QQ/Gmail，不能先用未配好的 `@inbox…`）。
2. 添加域名：`inbox.liubojian.com`。
3. 按 ImprovMX **DNS 记录** 页要求，在阿里云解析中为 **主机记录 `inbox`** 配置：
   - MX：`mx1.improvmx.com`（优先级 **10**）
   - MX：`mx2.improvmx.com`（优先级 **20**）
   - TXT（SPF）：`v=spf1 include:spf.improvmx.com ~all`
4. 解析请求来源选 **「默认」**。
5. 删除/避免与邮件推送冲突的旧 `inbox` MX。
6. ImprovMX 显示 **Active**（DNS 绿勾）。
7. 设置转发目标为 **主 QQ 邮箱**（亦曾出现绑定 Gmail 的界面记录；以你当前后台「别名」页实际目标为准，保持唯一清晰）。
8. 使用通配别名 `* @inbox.liubojian.com` → 转发到主邮箱。
9. **实测**：用 `@inbox.liubojian.com` 地址成功注册 **Claude** 账号，且流程中 **不需要手机号**。

官方 DNS 说明参考：[ImprovMX Generic DNS Configuration](https://improvmx.com/guides/generic-dns-configuration/)  
（文档示例 Host 为 `@` 时针对根域；子域 `inbox` 在阿里云主机记录填 **`inbox`**。）

### 4.2 通配含义（务必记住）

在开启 `*` 通配且 Active 的前提下：

- **前缀可任意**（字母数字等合理字符）
- **后缀固定** `inbox.liubojian.com`
- 发往该后缀的邮件，一般都会转到你设定的主邮箱（QQ）

示例：`reg01@inbox…`、`test999@inbox…`、`claude-xxx@inbox…` 均可。

无需每次登录 ImprovMX；**自动转发**。仅在换目标邮箱、改 DNS、域名过期或收不到信时再进后台。

### 4.3 免费与付费

- **收信转发**：免费层通常可用，不必为收验证码升级。
- **SMTP 凭证 / 向外发信**：高级功能，收码场景不需要。

---

## 5. 三个子域对照（以后查表用）

| 问题 | notice / mail | inbox |
|------|----------------|-------|
| 在哪创建的？ | 阿里云邮件推送 | ImprovMX |
| 干什么？ | 网站/系统 **发** 邮件 | **收** 别人发来的验证码 |
| DNS MX 指向？ | 阿里云邮件推送要求的 MX | `mx1/mx2.improvmx.com` |
| 要登录网页邮箱吗？ | 否 | 否（在 QQ 里看信） |
| 多个地址怎么来？ | 推送里建多个「发信地址」 | 通配 `*` 或添加别名 |
| 博客用哪个？ | **notice（已测）** | 一般不用来给博客 SMTP |

---

## 6. 日常使用清单

### 博客给用户发验证码

1. Typecho 插件 SMTP 使用 `@notice…`（或你指定的 mail 发信地址）。
2. 用户填写自己的 QQ/163 等 → 收码。

### 你在其它网站注册、收验证码

1. 邮箱填：`任意名@inbox.liubojian.com`
2. 打开 **主 QQ**（及垃圾箱）查收
3. 换号则换前缀即可

### 很久以后出问题先查

1. 域名 `liubojian.com` 是否过期  
2. 阿里云解析里 `notice` / `mail` / `inbox` 记录是否被误删  
3. ImprovMX 是否仍为 Active；别名通配是否还在  
4. 邮件推送发信地址 SMTP 密码是否更换、额度是否用尽  
5. 目标 QQ 是否还能登录  

---

## 7. 相关本地工程（博客）

- 路径：`/home/vdu/99.study_code/web_blogs/vdu_blogs/05-typecho`
- 启动：`bash start.sh`（Docker，端口常见 8085）
- 会员阅读分级等：`extras/plugins/MemberAccess`（与邮件无关，同属本站）
- 本邮件记录文档路径：`vdu_blogs/docs/email-domains-setup.md`

---

## 8. 安全与合规提醒（简记）

- 邮件推送、SMTP 密码、PAT 等曾存在本地 SQLite **明文**风险；公网部署前建议轮换密钥。
- 大量抛号、规避平台封禁可能违反对方条款；技术上转发只保证「信能转到 QQ」。
- 不要把含密钥的 `data/usr/*.db` 提交到公开仓库。

---

## 9. 一句话备忘

> **notice / mail = 阿里云推送发信；inbox = ImprovMX 通配收信转发到主 QQ。DNS 在阿里云分别配置，互不抢 MX。博客发码用 notice；自己多号收码用 *@inbox.liubojian.com。配通后 ImprovMX 自动转发，日常不用管。**

（文中具体主机名、优先级、SPF 字符串若与控制台不一致，以 **阿里云邮件推送配置页** 与 **ImprovMX DNS 记录页** 当前显示为准。）
