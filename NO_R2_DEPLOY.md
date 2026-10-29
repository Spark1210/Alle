# Alle 无 R2 版本部署说明

此版本针对无法开通 Cloudflare R2 的情况做了修改：

- 移除了 `wrangler.jsonc` 中的 R2 Bucket 绑定。
- 收件处理不再上传或保存附件；邮件正文、发件人、主题、收件信息等仍写入 D1。
- 邮件详情 API 始终返回 `attachments: []`，并移除了附件下载路由。
- 删除邮件或账号时不再调用 R2。
- 保留原有 D1 schema/migrations；不需要删除 `attachments` 表。

## 部署前必须做的事

1. 在自己的 Cloudflare 账户创建 D1 数据库：
   ```bash
   pnpm exec wrangler login
   pnpm exec wrangler d1 create alle
   ```
2. 将返回的数据库 ID 写入 `wrangler.jsonc` 的 `d1_databases[0].database_id`，替换 `REPLACE_WITH_YOUR_D1_DATABASE_ID`。
3. 生成/更新 Worker 类型：
   ```bash
   pnpm install
   pnpm exec wrangler types
   ```
4. 设置登录签名密钥（按项目原有配置）：
   ```bash
   pnpm exec wrangler secret put SECRET
   ```
   按提示输入自己生成的高强度随机字符串，不要提交到 Git。
5. 构建并部署：
   ```bash
   pnpm run build
   pnpm run deploy
   ```

## 功能限制

- **邮件附件被丢弃**：收到带附件的邮件时，只保存邮件正文和常规邮件字段，不保存附件内容或附件元数据。正文中引用 `cid:` 的内嵌图片不会显示。
- **不等于完整 SMTP 发信服务**：本修改只处理原项目已有功能；不会额外添加主动发信能力。
- **收件域名/路由仍需单独配置**：部署 Worker 和 D1 本身不会自动创建邮箱地址或自动配置 Cloudflare Email Routing。
- 本包尚未在连接到用户 Cloudflare 账户的真实环境中部署验证。部署前必须替换 D1 ID，并在自己的环境中完成构建和测试。
