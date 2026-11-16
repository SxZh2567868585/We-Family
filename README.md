# 我们一家 · We Family

移动端优先的家庭数字共享空间：把**聊天、待办、动态、记录本、共享记账**收进一个网址，
全家共用一个入口，还能用一句自然语言完成记账与待办创建。

- 前端：React 18 + Vite 5 + Tailwind CSS 3 + React Router 6 + Zustand + axios + dayjs
- 后端：PHP 8.1+（Laravel 10）+ MySQL 5.7 / 8.0 + Laravel Sanctum
- 形态：前后端分离 SPA，单域名同源部署；内容区最大宽度 480px，桌面端居中展示
- 许可：[MIT](LICENSE)

---

## 一、项目定位

市面上的家庭类 App 多数按「功能」切成多个孤立工具。本项目把它们合并成一个**以家庭为单位的数据空间**：

- 一个家庭 = 一个隔离的数据域，成员之间共享待办、动态、记录与账单；
- 每个人又能保留自己的私密待办、私密记录和私密账本；
- 用最少的操作完成记录：一句话 → AI 解析 → 结构化数据落库；
- 不依赖 App 商店，浏览器打开即用，适合部署在自有服务器上长期使用。

> 定位不是「通用 IM」或「企业协作」，而是**单家庭、小规模、低运维成本**的私人共享空间。

---

## 二、核心功能特性

| 模块 | 能力 |
| --- | --- |
| 账号 | 用户名 + 密码登录 / 注册；注册时创建家庭（成为管理员）或凭邀请码加入；Token 存 localStorage，7 天免登录 |
| 角色 | 家庭管理员：管理成员（邀请 / 重置密码 / 移除）、修改家庭信息；普通成员：全部业务功能 + 个人资料 |
| 聊天 | 单家庭群聊；文本 / 表情 / 图片（最多 9 张，JPG/PNG ≤5MB）/ 文件（≤10MB）/ 语音（≤60 秒）；连续消息合并头像；本人 2 分钟内可撤回；5 秒轮询增量拉取 |
| 待办 | 月历视图 + 每日待办数量；全家 / 私密双标签；重复规则（每天 / 每周 / 每月）+ 结束日期；勾选完成、编辑、删除；一句话 AI 创建 |
| 动态 | 文字 + 最多 9 图；点赞 / 取消；评论 / 删除本人评论；删除本人动态；下拉刷新 + 上拉分页 |
| 记录本 | 默认分类 + 自定义分类；标题 / 正文搜索；每条记录独立设置全家共享或仅自己可见；保留换行；删除二次确认 |
| 记账 | 家庭账本 / 个人私密账本；一句话 AI 记账（金额、类别、日期、备注）；解析失败转手动表单；环形图分类占比；本月 AI 用量展示 |
| 我的 | 头像昵称修改、修改密码、底部 Tab 顺序自定义、成员管理、退出登录（清理本地数据） |
| 安全 | 家庭级全局作用域、私密数据 SQL 层过滤、bcrypt 密码、Sanctum Token、上传二次校验、LIKE 通配符转义、统一 401 处理 |

---

## 三、适用场景

- 一个家庭（或两三人小组）需要一个**私有部署**的共享空间，不想把家庭数据交给第三方 SaaS；
- 有闲置服务器 / NAS，希望用单域名、单站点、单数据库完成部署；
- 需要「记账 / 待办」这类高频记录足够轻量，最好一句话完成；
- 希望代码可控、可二次开发（换对象存储、接实时推送、改主题）。

不适合：多租户 SaaS 运营、大规模并发聊天、需要 App 原生能力（推送、通讯录）的场景。

---

## 四、技术栈

| 层 | 技术 | 说明 |
| --- | --- | --- |
| 前端框架 | React 18 + Vite 5 | SPA，构建产物为静态文件 |
| 样式 | Tailwind CSS 3 | 全局暖橙 `#FF8C42` + 米白 `#FFF9F2`；`src/index.css` 定义卡片 / 按钮 / 输入框 |
| 路由 / 状态 | React Router 6 + Zustand | 状态持久化到 localStorage（登录态、Tab 顺序） |
| 请求 | axios | 拦截器统一注入 Bearer Token、解包 `{ code, message, data }`、401 清理登录态 |
| 时间 / 表单 | dayjs + react-hook-form | 聊天时间、月历、相对时间 |
| 后端框架 | Laravel 10 | RESTful API，全部路由在 `routes/api.php` |
| 鉴权 | Laravel Sanctum | Personal Access Token，默认 7 天有效 |
| 数据库 | MySQL 5.7.8+ / 8.0 | `utf8mb4`，JSON 字段存重复规则与 Tab 顺序 |
| 文件存储 | Laravel Filesystem | 默认本地 `public` 磁盘，预留 S3 / OSS 兼容配置 |
| AI | OpenAI Chat Completions 兼容协议 | 可接 OpenAI、通义千问兼容模式、DeepSeek 等；无密钥时走内置规则兜底 |

---

## 五、目录结构

```text
we-family/
├── README.md                     # 项目说明（本文档）
├── LICENSE                       # MIT 许可协议
├── docs/
│   ├── api.md                    # 接口清单与示例
│   ├── database.md               # 数据库设计与表关系
│   └── deployment.md             # 完整部署教程
├── deploy/
│   └── nginx.conf                # Nginx 单站点配置参考
│
├── backend/                      # Laravel 10 API
│   ├── app/
│   │   ├── Console/Kernel.php            # 计划任务（清理过期邀请码 / AI 日志）
│   │   ├── Exceptions/Handler.php        # 统一 JSON 错误响应（401/403/404/422/429）
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   │   ├── Controller.php
│   │   │   │   ├── HomeController.php    # 健康检查（Web 根路径）
│   │   │   │   └── Api/
│   │   │   │       ├── ApiController.php     # 统一响应封装
│   │   │   │       ├── AuthController.php    # 注册 / 登录 / 登出 / me
│   │   │   │       ├── ProfileController.php # 资料 / 密码 / Tab 顺序
│   │   │   │       ├── FamilyController.php  # 家庭、成员、邀请码、重置密码
│   │   │   │       ├── TodoController.php    # 待办
│   │   │   │       ├── PostController.php    # 动态 / 点赞 / 评论
│   │   │   │       ├── NoteController.php    # 记录本 / 分类
│   │   │   │       ├── BillController.php    # 记账 / 月度概览
│   │   │   │       ├── ChatController.php    # 聊天消息 / 撤回
│   │   │   │       ├── UploadController.php  # 通用上传
│   │   │   │       └── AiController.php      # AI 解析 / 用量
│   │   │   ├── Kernel.php
│   │   │   └── Middleware/
│   │   │       ├── EnsureFamilyMember.php    # 家庭上下文校验（+活跃时间）
│   │   │       └── EnsureFamilyAdmin.php     # 管理员校验
│   │   ├── Models/                       # 14 个模型 + 3 个全局作用域
│   │   │   └── Scopes/
│   │   │       ├── FamilyScope.php       # 家庭隔离
│   │   │       ├── VisibleScope.php      # 全家共享 / 仅自己可见
│   │   │       └── LedgerScope.php       # 家庭账本 / 私密账本
│   │   ├── Providers/
│   │   │   ├── AppServiceProvider.php    # 服务注册 + API 限流
│   │   │   └── RouteServiceProvider.php
│   │   ├── Services/
│   │   │   ├── AiService.php             # 大模型调用 + JSON 规范化 + 本地兜底
│   │   │   └── UploadService.php         # 上传校验 / 随机命名 / 分家庭目录
│   │   └── Support/
│   │       └── FinfoPolyfill.php         # 未安装 fileinfo 时的兜底实现
│   ├── bootstrap/app.php
│   ├── config/                           # app/auth/cors/database/filesystems/sanctum/services...
│   ├── database/
│   │   ├── migrations/                   # 16 个迁移（含 Sanctum Token 表）
│   │   └── seeders/DatabaseSeeder.php    # 演示家庭与账号
│   ├── lang/zh_CN/validation.php         # 中文校验提示
│   ├── public/index.php                  # PHP 入口
│   ├── routes/
│   │   ├── api.php                       # 全部业务接口
│   │   ├── web.php                       # 健康检查
│   │   └── console.php
│   ├── storage/                          # 上传文件、日志、缓存（需可写）
│   ├── .env.example                      # 本地环境变量模板
│   ├── .env.production.example           # 生产环境变量模板
│   └── composer.json
│
└── frontend/                     # React 18 SPA
    ├── src/
    │   ├── api/
    │   │   ├── client.js                 # axios 实例、Token 注入、错误归一
    │   │   └── index.js                  # 按业务分组的接口方法
    │   ├── components/
    │   │   ├── layout/                   # AppShell / AuthLayout / TabBar
    │   │   └── ui/                       # Button / Input / Sheet / Toast / Confirm
    │   │                                 # Avatar / Segmented / Skeleton / EmptyState
    │   │                                 # ImageUploader / EmojiPicker / DonutChart
    │   ├── pages/                        # Login / Register / Chat / Todo / Feed
    │   │                                 # Note / Bill / Profile
    │   ├── store/                        # auth / nav / ui（Zustand）
    │   ├── utils/                        # date / format / asset / cx
    │   ├── App.jsx                       # 路由表
    │   ├── main.jsx
    │   └── index.css                     # Tailwind + 设计系统
    ├── index.html
    ├── tailwind.config.js
    ├── vite.config.js                    # 开发代理 /api、/storage
    ├── postcss.config.js
    ├── package.json
    └── .env.example
```

---

## 六、核心模块说明

### 1. 家庭隔离与私密数据（`app/Models/Scopes`）

- 所有业务表都带 `family_id`，`FamilyScope` 作为 Eloquent 全局作用域，自动追加 `family_id = 当前用户家庭`；
- 写入时由模型 `creating` 事件强制回填 `family_id / user_id`，前端伪造无效；
- 待办、记录用 `VisibleScope`（`visibility=family OR 本人`），账单用 `LedgerScope`（`ledger=family OR 本人`），
  越权请求在 SQL 层就查不到数据，而不是前端隐藏。

### 2. AI 解析（`AiService` + `AiController`）

```text
前端一句话
  → POST /api/ai/parse-bill（或 /parse-todo）
  → AiService 调用兼容 OpenAI 协议的服务
  → 提取并规范化 JSON（金额、类别白名单、日期、时间、重复规则）
  → 失败且开启 AI_LOCAL_FALLBACK 时走内置中文规则
  → 写 ai_logs（家庭、场景、是否成功、模式、耗时）
```

未配置 `AI_API_KEY` 也能体验：「早饭 8 元」「昨天打车 26 块」「明天下午三点接孩子」等常见表达由本地规则解析。

### 3. 上传（`UploadService`）

- 校验真实 MIME、图片额外 `getimagesize()`、大小上限；
- UUID 随机文件名 + `uploads/{familyId}/{kind}/{Ym}` 分目录，扩展名白名单，杜绝 `.php` 落盘；
- 默认写本地 `public` 磁盘，切换 `FILESYSTEM_DISK=s3` 即可换对象存储，业务代码无需改动；
- 服务器未安装 `fileinfo` 扩展时，由 `app/Support/FinfoPolyfill.php` 兜底，保证上传不因环境缺失而 500。

### 4. 前端状态与请求

- `store/auth.js`：token、用户信息、过期时间，持久化到 localStorage；
- `api/client.js`：请求注入 Bearer Token，响应解包统一结构，401 自动清理登录态并跳转 `/login`；
- `store/nav.js`：底部 Tab 顺序持久化；`store/ui.js`：Toast 与二次确认。

---

## 七、快速开始

### 1. 环境要求

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| PHP | 8.1+（推荐 8.2 / 8.3） | 扩展：`openssl`、`pdo_mysql`、`mbstring`、`tokenizer`、`xml`、`ctype`、`json`、`bcmath`、`curl`、`fileinfo`（推荐） |
| Composer | 2.x | 依赖安装 |
| MySQL | 5.7.8+ / 8.0 | `utf8mb4`，需支持 JSON 字段 |
| Node.js | 18+（推荐 20 LTS） | Vite 5 要求 |
| npm | 9+ | 或 pnpm / yarn |

> 只想本地看效果、暂时没有 MySQL：把 `DB_CONNECTION=sqlite`、
> `DB_DATABASE=/绝对路径/database/database.sqlite`，并先创建空文件 `backend/database/database.sqlite`。

### 2. 启动后端

```bash
cd backend

composer install
cp .env.example .env          # Windows: copy .env.example .env
php artisan key:generate

# 创建数据库（示例）
mysql -u root -p -e "CREATE DATABASE we_family CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 按实际情况修改 .env 中的 DB_HOST / DB_DATABASE / DB_USERNAME / DB_PASSWORD
php artisan migrate --seed    # 建表 + 演示数据
php artisan storage:link      # 建立公开磁盘软链接（图片 / 文件 / 语音访问必需）
php artisan serve             # http://localhost:8000
```

演示账号（仅由 `--seed` 创建）：

| 账号 | 密码 | 角色 |
| --- | --- | --- |
| `demo` | `123456` | 家庭管理员（小满之家） |
| `mama` | `123456` | 普通成员 |

> 重来一遍：`php artisan migrate:fresh --seed`（会清空数据）。

### 3. 启动前端

```bash
cd frontend
npm install
npm run dev                   # http://localhost:5173
```

浏览器打开 `http://localhost:5173`，默认通过 Vite 代理访问 `/api`，无需额外配置。
手机调试：与电脑同一局域网，访问 `http://<电脑IP>:5173`（Vite 已监听 `0.0.0.0`）。

### 4. 五分钟验收路径

1. `demo` 登录 → 底部 6 个 Tab：聊天 / 待办 / 动态 / 记录本 / 记账 / 我的。
2. 「我的」里把「记账」上移 → 保存后底部导航立即变化，刷新仍保持。
3. 「待办」→ AI 输入「明天下午三点接孩子放学」→ 智能填写 → 创建。
4. 「记账」→ 输入「昨天打车 26 块」→ 解析 → 确认 → 列表、环形图、AI 次数同步更新。
5. 「动态」→ 发布文字 + 图片 → 点赞、评论、删除本人评论。
6. 「聊天」→ 发送文本 / 表情 / 图片 / 文件；HTTPS 或 localhost 下可录语音。
7. 「记录本」→ 新建「全家共享」和「仅自己可见」各一条 → 换 `mama` 登录只看到共享那条。
8. 「我的」→ 管理员生成邀请码 → 用新账号 + 邀请码加入家庭。

---

## 八、配置说明

完整变量见 [`backend/.env.example`](backend/.env.example)（逐项注释）。关键项：

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `APP_ENV` / `APP_DEBUG` | local / true | 生产必须 `production` / `false` |
| `APP_URL` | http://localhost:8000 | 生成图片 / 文件 URL 的根地址，部署后改成真实域名 |
| `APP_TIMEZONE` | Asia/Shanghai | AI 解析「今天 / 昨天 / 本月」的基准时区 |
| `APP_KEY` | 空 | `php artisan key:generate` 生成，**丢失会导致旧 Token 与加密数据失效** |
| `DB_*` | — | 数据库连接 |
| `SANCTUM_EXPIRATION` | 10080 | Token 有效期（分钟） |
| `CORS_ALLOWED_ORIGINS` | localhost:5173 | 允许跨域的前端来源（逗号分隔），生产写死域名 |
| `FILESYSTEM_DISK` | public | 本地公开磁盘；切对象存储时改 `s3` |
| `UPLOAD_IMAGE_MAX_KB` / `UPLOAD_FILE_MAX_KB` | 5120 / 10240 | 图片 / 文件大小上限 |
| `AI_BASE_URL` / `AI_MODEL` / `AI_API_KEY` | OpenAI / gpt-4o-mini / 空 | 兼容 OpenAI 协议的服务；密钥只放 `.env` |
| `AI_LOCAL_FALLBACK` | true | 模型不可用时是否本地规则兜底 |
| `INVITE_EXPIRE_HOURS` | 72 | 邀请码有效期 |
| `MESSAGE_RECALL_SECONDS` | 120 | 消息可撤回时间窗口 |

前端仅一个变量：

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `VITE_API_BASE_URL` | `/api` | 开发走 Vite 代理；前后端分域时填完整地址，如 `https://api.example.com/api` |

> 修改 `backend/.env` 后需重启 `php artisan serve`，生产需执行 `php artisan optimize:clear` 再重建缓存。

---

## 九、生产部署

完整步骤（环境依赖、安装、配置、启动、Nginx、HTTPS、计划任务、FAQ）见
[`docs/deployment.md`](docs/deployment.md)，Nginx 参考 [`deploy/nginx.conf`](deploy/nginx.conf)。

最小流程：

```bash
# 后端
cd backend
composer install --no-dev --optimize-autoloader
cp .env.production.example .env      # 修改域名、数据库、APP_KEY
php artisan key:generate --force
php artisan migrate --force
php artisan storage:link
php artisan config:cache && php artisan route:cache && php artisan view:cache
chown -R www-data:www-data storage bootstrap/cache
chmod -R 775 storage bootstrap/cache

# 前端
cd ../frontend
npm ci && npm run build
cp -r dist/. ../backend/public/      # 保留 public/index.php 与 .htaccess
```

Nginx 要点：

- `root` 指向 `backend/public`，`index index.html index.php;`（`index.html` 必须在 `index.php` 之前）；
- `location /` 使用 `try_files $uri $uri/ /index.html`（BrowserRouter 刷新不 404）；
- `location ^~ /api/` 交给 `index.php`；`location ^~ /storage/` alias 到 `storage/app/public` 并禁止执行脚本；
- `client_max_body_size` 与 PHP `upload_max_filesize` / `post_max_size` 同步调大（聊天一次最多 9 张图）；
- 生产必须 HTTPS：录音需要安全上下文，Token 与家庭数据不应明文传输。

计划任务（清理过期邀请码、旧 AI 日志）：

```cron
* * * * * cd /var/www/we-family/backend && php artisan schedule:run >> /dev/null 2>&1
```

---

## 十、安全设计

1. **家庭隔离**：`FamilyScope` 强制 `family_id`；私密数据由 `VisibleScope` / `LedgerScope` 在 SQL 层过滤。
2. **鉴权**：业务接口统一 `auth:sanctum + family`；Token 过期统一 401，前端清理本地态并跳登录页。
3. **限流**：登录 / 注册 10 次/分钟，AI 30 次/分钟，全局 API 120 次/分钟。
4. **密码**：bcrypt 哈希；管理员重置密码后吊销该成员全部 Token，并强制下次登录修改密码。
5. **上传**：真实 MIME + `getimagesize()` + 大小 + 扩展名白名单 + 随机文件名，禁止脚本落盘。
6. **输入**：全部写接口 `validate()`；搜索转义 LIKE 通配符；前端不使用 `dangerouslySetInnerHTML`；
   Eloquent 参数绑定防注入。
7. **消息撤回**：仅本人、2 分钟内，服务端二次判定，撤回后不再返回内容与文件地址。

> 部署时请关闭 `APP_DEBUG`，不要把 `.env`、`storage/logs`、`storage/app/public/uploads` 提交到仓库。

---

## 十一、常见问题

| 现象 | 排查 |
| --- | --- |
| 接口 401 循环跳登录 | Token 过期（默认 7 天）或 `APP_KEY` 变更；重新登录 |
| 图片 / 语音 404 | 未执行 `php artisan storage:link`，或 `APP_URL` 与实际访问地址不一致 |
| 上传 413 | Nginx `client_max_body_size`、PHP `upload_max_filesize` / `post_max_size` 太小 |
| 上传 500 且日志提示 `Class "finfo" not found` | PHP 未装 `fileinfo` 扩展；本项目已有兜底实现，仍建议在 PHP 扩展中安装 fileinfo |
| 跨域被拦截 | `CORS_ALLOWED_ORIGINS` 未包含前端域名；改完执行 `php artisan optimize:clear` |
| 无法录音 | 浏览器要求 HTTPS 或 localhost；部分内置浏览器兼容性有限 |
| AI 总是 fallback | `AI_API_KEY` 为空、`AI_BASE_URL` 不可达或模型名错误，查看 `storage/logs/laravel.log` |
| 迁移报 JSON 不支持 | MySQL 5.7.8 以下不支持 JSON 字段，请升级 |
| `composer install` 内存不足 | `COMPOSER_MEMORY_LIMIT=-1 composer install` |
| 前端刷新 404 | Nginx 缺少 `try_files ... /index.html` |
| 首页显示 JSON 健康检查而不是页面 | `index` 指令顺序错误，`index.html` 必须在 `index.php` 之前 |

---

## 十二、已知边界

- 聊天为 5 秒轮询，不是 WebSocket；单家庭规模足够，后续可平滑替换为广播。
- 语音格式由浏览器决定（Chrome / Edge 为 `webm`，Safari 可能为 `mp4`），后端已放宽对应 MIME。
- 记录本搜索基于 `LIKE`，适合单家庭万级数据；超大数据量建议全文索引。
- 被移除成员的账号保留（`status=removed`），用户名仍占全局唯一约束，避免历史数据错乱。
- 不提供自助找回密码，统一由家庭管理员重置。
- 内置规则兜底只覆盖常见中文表达，复杂语句建议配置大模型密钥。

---

## 十三、贡献指南

1. Fork 本仓库并创建特性分支：`git checkout -b feature/your-feature`；
2. 后端改动请保持 `php artisan migrate` 可重复执行，前端改动请保持移动端 480px 布局；
3. 提交前自测：`php artisan migrate:fresh --seed` + `npm run build`；
4. 提交 Pull Request，说明改动目的、影响范围与验证方式。

提交代码前请确认没有携带 `.env`、密钥、证书、日志与上传文件。

---

## 十四、许可证

本项目基于 [MIT License](LICENSE) 开源，可自由用于个人与商业项目。
