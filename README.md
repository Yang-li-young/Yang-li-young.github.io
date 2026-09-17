# Hexo 博客站点（lixiaoyao/V1）
Creating a new branch is quick.

基于 [Hexo 8.1.2](https://hexo.io/) 的静态博客项目，使用 **pnpm** 管理依赖，默认启用 **Landscape** 主题。目前为初始化状态，仅包含 Hexo 自带的示例文章《Hello World》。

## 目录结构总览

```text
V1/
├── .github/                # GitHub 相关配置
│   └── dependabot.yml      # Dependabot 自动依赖更新配置
├── node_modules/           # 依赖包目录（pnpm 管理，无需手动修改）
├── scaffolds/              # 文章模板目录
│   ├── post.md             # 新文章（post）模板
│   ├── draft.md            # 草稿（draft）模板
│   └── page.md             # 自定义页面（page）模板
├── source/                 # 站点内容源文件目录
│   └── _posts/             # 博客文章目录
│       └── hello-world.md  # Hexo 初始化自带的示例文章
├── themes/                 # 本地主题目录（当前为空）
│   └── .gitkeep            # 占位文件，用于让 Git 保留空的 themes 目录
├── _config.yml             # Hexo 站点主配置文件
├── _config.landscape.yml   # Landscape 主题配置覆盖文件（当前为空）
├── .gitignore              # Git 忽略规则
├── db.json                 # Hexo 本地数据库缓存（自动生成，勿手动编辑）
├── package.json            # npm 包定义与脚本命令
└── pnpm-lock.yaml          # pnpm 依赖锁定文件（自动生成）
```

## 文件详细说明

### 配置类文件

#### `_config.yml` — Hexo 站点主配置文件

Hexo 的核心配置，控制站点全局行为，主要配置段包括：

| 配置段 | 作用 |
|--------|------|
| `Site` | 站点基本信息：标题（`title`）、副标题、描述、关键词、作者、语言、时区 |
| `URL` | 站点地址与文章永久链接格式（当前为 `:year/:month/:day/:title/`） |
| `Directory` | 各类资源的目录名（源文件、生成产物、标签页、归档页等） |
| `Writing` | 写作相关：新文章文件名格式、默认布局、代码高亮（highlight.js）设置 |
| `index_generator` | 首页设置：每页显示 10 篇文章，按日期倒序排列 |
| `Date / Time format` | 日期时间显示格式 |
| `Pagination` | 分页设置：每页 10 篇 |
| `theme` | 当前使用的主题：`landscape` |
| `deploy` | 部署配置（当前为空，未配置部署方式） |

> ⚠️ 注意：当前站点信息仍是默认值（标题为 "Hexo"、作者为 "John Doe"、URL 为 `http://example.com`），使用前请修改为自己的信息。

#### `_config.landscape.yml` — 主题配置覆盖文件

Hexo 支持的独立主题配置机制（`_config.[主题名].yml`）。此文件中的配置会**覆盖**主题包（`node_modules/hexo-theme-landscape/_config.yml`）内的同名配置，好处是升级主题时自定义配置不会丢失。当前文件为空（0 字节），即完全使用主题默认配置。

#### `.gitignore` — Git 忽略规则

声明不纳入版本控制的文件：

- `db.json` — Hexo 本地缓存
- `node_modules/` — 依赖包
- `public/` — 生成的静态站点产物
- `.deploy*/` — 部署临时文件
- `.DS_Store` / `Thumbs.db` — 系统文件（macOS / Windows）
- `*.log` — 日志文件
- `_multiconfig.yml` — 多配置合并的临时文件

#### `package.json` — npm 包定义

- **`scripts`**：定义了四个常用命令别名：
  - `pnpm build` → `hexo generate`（生成静态文件）
  - `pnpm clean` → `hexo clean`（清理缓存与产物）
  - `pnpm deploy` → `hexo deploy`（部署站点）
  - `pnpm server` → `hexo server`（启动本地预览服务器）
- **`dependencies`**：列出 Hexo 核心及各插件的版本要求，包括：
  - `hexo` — Hexo 核心（^8.0.0）
  - `hexo-generator-archive` / `hexo-generator-category` / `hexo-generator-index` / `hexo-generator-tag` — 分别生成归档页、分类页、首页索引、标签页
  - `hexo-renderer-ejs` / `hexo-renderer-marked` / `hexo-renderer-stylus` — 渲染器：EJS 模板、Markdown、Stylus 样式
  - `hexo-server` — 本地预览服务器
  - `hexo-theme-landscape` — Landscape 主题包
- **`hexo.version`**：记录站点初始化时使用的 Hexo 版本（8.1.2）

#### `pnpm-lock.yaml` — pnpm 依赖锁定文件

pnpm 自动生成，锁定全部依赖的精确版本与下载地址，保证团队成员或 CI 环境安装出完全一致的依赖树。**不要手动编辑**，由 pnpm 自动维护。

### `.github/` — GitHub 配置目录

#### `.github/dependabot.yml` — Dependabot 配置

配置 GitHub Dependabot **每天**（`interval: daily`）自动检查根目录（`/`）下 npm 生态的依赖更新，发现新版本时最多自动开 20 个（`open-pull-requests-limit: 20`）升级 Pull Request，保持依赖安全与新鲜。

### `scaffolds/` — 模板目录

执行 `hexo new` 命令创建新内容时使用的 Front-matter 模板（`{{ title }}`、`{{ date }}` 为占位符，创建时自动替换）：

| 文件 | 对应命令 | 模板内容 |
|------|----------|----------|
| `post.md` | `hexo new "标题"` | 预填 `title`、`date`、`tags` 三个字段 |
| `draft.md` | `hexo new draft "标题"` | 预填 `title`、`tags` 字段（草稿无日期） |
| `page.md` | `hexo new page "名称"` | 预填 `title`、`date` 字段 |

### `source/` — 内容源目录

Hexo 站点的内容存放处。除下划线开头的文件夹外，`source/` 下的所有文件/文件夹都会被原样复制到生成的静态站点中（Markdown 与 Stylus 文件会被渲染）。

#### `source/_posts/hello-world.md` — 示例文章

Hexo 初始化时自动生成的第一篇文章，标题为《Hello World》，内容是 Hexo 官方快速入门指南（新建文章、启动服务器、生成静态文件、部署四个基本操作）。**这是目前站点唯一的一篇文章**。

以 `_` 开头的文件夹（如 `_posts`）为特殊资源目录，不会被直接复制，而是由 Hexo 处理后输出。

### `themes/` — 本地主题目录

用于存放通过 `git clone` 方式安装的主题。当前目录为空，实际使用的 Landscape 主题是从 npm 安装、位于 `node_modules/hexo-theme-landscape/` 中。若将来想自定义主题，可克隆一份到此目录并修改 `_config.yml` 的 `theme` 字段。

`.gitkeep` 是约定俗成的占位文件，唯一作用是让 Git 跟踪空目录（Git 本身不跟踪空目录）。

### 自动生成的文件（无需手动维护）

#### `db.json` — Hexo 本地数据库

Hexo 使用 [Warehouse](https://github.com/hexojs/warehouse) 模块生成的 JSON 数据库，缓存了站点全部数据：文章（含渲染后的 HTML）、资源、分类、标签以及各源文件的哈希值，用于加速增量构建。它已列入 `.gitignore`，不要提交也不要手动编辑；若出现数据异常，运行 `hexo clean` 删除后重新生成即可。

#### `node_modules/` — 依赖包目录

pnpm 安装的全部依赖（Hexo 核心、渲染器、生成器插件、Landscape 主题等约 7000+ 文件），其中 `.pnpm/` 采用符号链接实现包共享存储。此目录由 pnpm 管理，不应手动修改或提交。

## 常用命令

```bash
# 安装依赖
pnpm install

# 新建文章（使用 scaffolds/post.md 模板）
pnpm exec hexo new "我的新文章"

# 启动本地预览（默认 http://localhost:4000）
pnpm server

# 生成静态文件到 public/ 目录
pnpm build

# 清理缓存（db.json）与生成产物（public/）
pnpm clean

# 部署（需先在 _config.yml 中配置 deploy 段）
pnpm deploy
```

## 参考

- [Hexo 官方文档](https://hexo.io/docs/)
- [Landscape 主题](https://github.com/hexojs/hexo-theme-landscape)
- [pnpm 文档](https://pnpm.io/)