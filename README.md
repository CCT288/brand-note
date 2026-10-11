# BRAND NOTE

给商家图片和视频关键截图写小红书发布文字的个人工作台。Next.js App Router、TypeScript、Tailwind、shadcn/ui、Supabase，MIT 许可证。

## 使用流程

在“品牌”配置品牌事实、表达风格和六个默认风格滑杆；在“写文案”填本次素材说明，可上传最多六张 PNG/JPEG/WebP（每张小于 5 MB）。“快速”保持简洁，“详细”展开细节和本次滑杆。视频请先选关键截图，V1 不解析整段视频。

服务逐步准备品牌信息、理解素材、参考历史内容、构思至少六个方向、筛选三个方向、撰写文案、检查文案。每步单独请求并保存进度；失败时保留输入，可点击“继续生成”。JSON 格式最多修复两次，质量检查最多重写一次，再失败明确报错。内部检查内容不返回浏览器。

三个方向全部可编辑。“换标题”提供三个标题建议；“重写正文”保留标题和方向；自由反馈可指定标题、正文或一段。建议只有点击“应用建议”才生效，受影响字段已被手动修改时拒绝覆盖。“采用这个方向”打开最终编辑；“保存到资料库”才归档并参与下次历史。同一篇重复保存不会新增文章，AI 初稿、修改建议和最终稿分开保留。

历史使用当前品牌最近 30 篇已归档内容，优先最终稿；最近五篇类型集中时仅轻量提示。长期偏好至少有三篇独立已归档内容支持才记录，未采用的建议与一次活动反馈不成为长期规则。

## 当前验收

线上地址：[BRAND NOTE](https://brand-note-buzz.netlify.app)，源码采用 MIT 许可证，品牌资料、素材、历史和凭据不进入公开仓库。

已在 Netlify 实际验证 Plus 凭据刷新及轮换、账号可用模型、中文与结构化输出、图片输入，以及完整七阶段文本生成、局部修改和云端保存。具体证据与未验证项见 `验收记录.md`。真实手机与关机后访问、商家文案质量仍需人工验收，不能用浏览器尺寸模拟代替。

## 获取源码

公开仓库提供 [完整源码 ZIP](https://github.com/CCT288/brand-note/raw/refs/heads/main/BRAND-NOTE-V1-Source.zip)。下载并解压，进入 `brand-note/` 后再运行下面的命令。包内含应用、数据库迁移、配置示例、测试和 MIT 许可证；不含私有运行资料。

## 本机启动

Node.js 24 与 pnpm，依赖已锁定。

```sh
pnpm install --frozen-lockfile
pnpm dev
```

打开 `http://127.0.0.1:3010/settings`。已安装的 Mac 也可双击 `启动 BRAND NOTE.command`。本机模式只监听回环地址，数据存在 `.runtime/`。不配置 Supabase 时不能跨设备同步。个人品牌样例保存在私有运行目录，不在源码中。

## Plus 连接

在“设置”点“连接 ChatGPT”，本人完成 OpenAI 官方登录与计划使用授权，保持额外付费额度关闭。点击“获取可用模型”后选择实际返回的模型，添加测试图片并点“测试真实生成”。只接受成功终止事件和通过结构校验的结果；没有付费 API key 回退。

官方协议说明：[Sign in](https://developers.openai.com/siwc/token-sharing-open-source/sign-in)、[模型与推理](https://developers.openai.com/siwc/token-sharing-open-source/models-and-inference)、[独立远程 host](https://developers.openai.com/siwc/token-sharing-open-source/self-hosted-vms)。自托管 VM 的说明不能直接证明 Netlify 函数环境可用，需实测。

## Supabase 与部署

1. Supabase Free 项目按文件名顺序运行 `supabase/migrations/` 中的 SQL。迁移创建八张核心表及服务端加密记录与刷新锁、RLS、私有 `brand-assets` bucket。
2. 在 Supabase Auth 手动创建唯一应用用户，关闭公开注册；将该 Auth UUID 写入 `private.workspace_owner`，并配置 `APP_OWNER_ID`。Supabase 控制台账号不等同应用账号。
3. 参照 `.env.example` 配置项目 URL、publishable key、仅服务端 secret key、APP_URL。云端必须设置 APP_DEPLOYMENT=cloud、独立持久 BRAND_NOTE_HOST_ID 与随机 32 字节 base64 加密密钥。
4. 停止本机服务后，用 `node --env-file=.runtime/cloud.env node_modules/tsx/dist/cli.mjs scripts/transfer-connection.ts` 将授权迁移到 `ai_records`。私有 cloud.env 必须使用云端 host 和密钥；脚本回读校验后将本机凭据改名留存，不继续使用。用 AES-256-GCM 加密且绑定 host/记录名。云端通过原子数据库租约锁串行刷新；不在函数临时文件系统保存凭据。部署后重新做真实连接测试，不能复制本机验证报告充当通过。
5. Netlify Free 使用 `netlify.toml` 和官方 Next.js adapter，保持免费计划，不启用充值或付费附加服务。达到额度可能暂停。网站以唯一应用账号登录，数据不会因为代码开源而公开。

## 安全与检查

每个 API 验证账号归属，写请求额外检查来源；RLS 同时限制唯一账号和行 user_id。素材必须登录读取，云端访问链接只短时有效。环境配置不完整时云端拒绝访问。授权撤销、额度错误、中断与版本冲突保留输入和草稿。

禁止公开 `.runtime/`、`.env*`、个人截图、真实品牌资料、素材、历史或凭据。构建文件追踪排除私有运行目录与环境文件。开发日志关闭请求 URL，避免记录授权回调代码。

```sh
pnpm typecheck
pnpm lint
pnpm test
pnpm build
```

自动测试覆盖 OAuth/PKCE、身份与来源校验、流式成功/失败边界、加密防篡改、真实 PostgreSQL 迁移/RLS/原稿保护/幂等保存/刷新锁，以及历史和学习阈值。测试使用虚构资料，不代替线上实际操作。
