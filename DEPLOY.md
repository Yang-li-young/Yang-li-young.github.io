# 博客个性化部署指南

本文档面向本仓库的**所有者自己**，说明如何把这套 Hexo 模板改造成你的个性化博客并部署上线。

## 仓库现状速览

| 项目 | 现状 | 目标 |
|------|------|------|
| 站点框架 | Hexo 8.1.2（要求 Node ≥ 20.19，本机 v24.11.1 满足） | 保持 |
| 包管理器 | pnpm 10.15.1 | 保持 |
| 远程仓库 | `https://github.com/Yang-li-young/Yang-li-young.github.io.git`（GitHub Pages **用户站点**，部署后地址为 `https://yang-li-young.github.io`） | 保持 |
| 分支模型 | `dev` → PR → `main`（main 为默认分支） | 保持，main 触发自动部署 |
| 主题 | `_config.yml` 仍是 `theme: landscape`（npm 安装） | 切换到已克隆的 **hexo-theme-matery** |
| 文章 | 仅 `source/_posts/hello-world.md` 一篇 | 写自己的内容 |
| 部署方式 | `deploy` 配置为空，`.github/` 仅有 Dependabot | 增加 GitHub Actions 自动构建发布 |

> ⚠️ **当前最要紧的一件事**：`themes/hexo-theme-matery/` 是通过 `git clone` 下载的，里面还带着它自己的 `.git` 目录，且尚未纳入版本控制。直接 `git add` 会把它记录成一个"内嵌仓库"（gitlink），推上 GitHub 后 Actions 检出时会得到**空目录**，构建必然失败。请先完成[第 2 节](#二引入-matery-主题关键步骤)。

---

## 一、环境准备

已在本机跑通过 `pnpm install` 的话可跳过本节。

```powershell
# 1. 安装 Node.js LTS（必须 ≥ 20.19，建议 22/24 LTS）：https://nodejs.org/
node -v

# 2. 安装 pnpm
npm install -g pnpm
pnpm -v

# 3. Git（已安装并配置好 GitHub 凭据即可）
git -C D:\Code\lixiaoyao\V1 remote -v   # 应显示 origin 指向你的 github.io 仓库
```

## 二、引入 Matery 主题（关键步骤）

### 2.1 删除主题内嵌的 `.git`

```powershell
# PowerShell：彻底删除主题目录里嵌套的 git 元数据（不是删除主题！）
Remove-Item -Recurse -Force D:\Code\lixiaoyao\V1\themes\hexo-theme-matery\.git
```

删完后 `themes/hexo-theme-matery` 就变成普通文件夹，可以正常提交进你的仓库（约几 MB，可接受）。

> 为什么要这么做：不删 `.git` 的话，`git add` 只会记录一个指向原仓库的引用而非文件内容；GitHub 上点开该目录是空的，Actions 构建时主题缺失直接报错。
> 想跟随主题上游更新的代价：之后升级主题需要手动下载新版 zip 解压覆盖（覆盖前备份你自己改过的文件），或者改用 git submodule 管理（不推荐，新手容易翻车）。

### 2.2 卸载 Landscape 主题相关内容

```powershell
# 移除 npm 版 Landscape 主题依赖
pnpm remove hexo-theme-landscape

# 删除 Landscape 的主题覆盖配置文件（当前为空文件，留着也不报错，但容易混淆）
Remove-Item D:\Code\lixiaoyao\V1\_config.landscape.yml
```

### 2.3 切换主题

编辑根目录 `_config.yml`：

```yaml
# Extensions 段
theme: hexo-theme-matery
```

> 主题名必须与 `themes/` 下的文件夹名一致。如果你嫌名字长，也可以把文件夹改名为 `matery`，然后把 `theme` 值和下文的覆盖配置文件名相应改成 `matery`。

### 2.4 创建主题覆盖配置文件（强烈推荐）

Hexo 支持站点根目录下 `_config.[主题名].yml` 覆盖机制：该文件中的配置会**合并覆盖**主题包内的同名配置。好处是将来升级主题（整目录覆盖）时，你的个性化配置不会丢。

```powershell
# 把主题自带配置整体复制一份作为"你的"配置文件，之后只改这一份
Copy-Item D:\Code\lixiaoyao\V1\themes\hexo-theme-matery\_config.yml D:\Code\lixiaoyao\V1\_config.hexo-theme-matery.yml
```

此后**所有主题相关的个性化都改 `_config.hexo-theme-matery.yml`**，不要直接改 `themes/hexo-theme-matery/_config.yml`（除非要改主题源码，如页脚模板）。

### 2.5 验证主题切换成功

```powershell
pnpm clean
pnpm server   # 浏览器打开 http://localhost:4000，应看到 Material Design 风格的新首页
```

## 三、个性化站点主配置（`_config.yml`）

按下表逐项修改根目录 `_config.yml`（只列需要动的字段）：

| 字段 | 改为 | 说明 |
|------|------|------|
| `title` | 你的博客名（如 `小遥的自留地`） | 显示在标题栏、首页 banner |
| `subtitle` | 一句话副标题 | banner 下方打字机文案 |
| `description` | 80 字以内的站点描述 | **SEO 关键字段** |
| `keywords` | `博客,技术,生活` 等 | SEO |
| `author` | 你的名字 | 署名 |
| `language` | `zh-CN` | 中文界面（主题内置中文语言包） |
| `timezone` | `Asia/Shanghai` | 时间显示时区 |
| `url` | `https://yang-li-young.github.io` | **必须改**，影响站点内所有绝对链接、RSS |
| `permalink` | 保持 `:year/:month/:day/:title/`（中文标题建议配合拼音插件，见第六节） | 文章永久链接格式 |
| `per_page` 和 `index_generator.per_page` | `12`（两处都改） | 建议为 6 的倍数，瀑布流列表才整齐 |
| `post_asset_folder` | `true` | 可选；`hexo new` 时自动为每篇文章建同名图片文件夹 |
| `syntax_highlighter` | `prismjs` | 见下方说明 |
| `default_category` | `tech` 之类 | 可选，默认 `uncategorized` |

**代码高亮的重要说明**：主题自带的 README 写的是 `highlight.enable: false` + `prismjs.enable: true`，那是 **Hexo 5 的旧写法**。本仓库是 Hexo 8，只需要一行：

```yaml
syntax_highlighter: prismjs
```

（当前值是 `highlight.js`，务必改掉，否则文章代码块样式与主题不匹配。）

修改后的开头几行示例：

```yaml
# Site
title: 小遥的自留地
subtitle: 记录技术与生活
description: 一个关于编程、折腾与日常的个人博客
keywords: 博客,前端,编程
author: 小遥
language: zh-CN
timezone: Asia/Shanghai

# URL
url: https://yang-li-young.github.io
```

## 四、主题配置个性化（`_config.hexo-theme-matery.yml`）

按功能模块逐个介绍**最值得改**的配置段（行号以复制出来的文件为准，顺序一致）。没有提到的段落保持默认即可。

### 4.1 菜单导航 `menu`

```yaml
menu:
  首页:
    url: /
    icon: fas fa-home
  标签:
    url: /tags
    icon: fas fa-tags
  分类:
    url: /categories
    icon: fas fa-bookmark
  归档:
    url: /archives
    icon: fas fa-archive
  关于:
    url: /about
    icon: fas fa-user-circle
  # 留言板和友链按需取消注释：
  # Contact / Friends ...
```

- 菜单名可用中文，图标在 [Font Awesome](https://fontawesome.com/icons) 搜索（主题用 5.x/6.x，`fas`/`fab` 前缀）。
- 菜单里的每个 URL 都需要真实页面存在（见第五节创建 categories/tags/about 等页），否则点进去是 404。
- 支持二级菜单，写法见文件内注释的 `children` 示例。

### 4.2 站点图标与 LOGO

```yaml
favicon: /favicon.png     # 浏览器标签页图标
logo: /medias/logo.png    # 导航栏 LOGO
```

**放文件的位置**：路径是相对站点根的，把你自己的图片放到**你博客的** `source/` 下即可，同名文件会覆盖主题自带的：

```
source/
├── favicon.png
└── medias/
    ├── logo.png
    ├── avatar.jpg            # 关于页头像（4.6 节 profile 用）
    └── banner/              # 首页 banner 大图，见 4.3
        ├── 0.jpg            # 兜底图（当天无对应编号时显示）
        ├── 1.jpg            # 周一
        ├── ...
        └── 6.jpg            # 周日
```

> 4.3 banner 机制：主题首页大图按**今天是星期几（0=周日…6=周六）**自动轮换 `medias/banner/<编号>.jpg`。主题自带 7 张图，想换成自己的，就在 `source/medias/banner/` 放同名 `0.jpg`~`6.jpg` 覆盖。`jsDelivr.url` 保持为空即可（那是为了把图片走 CDN 加速用的，本地/仓库路径更省事）。

### 4.3 首页横幅按钮与社交链接

```yaml
# banner 上第二个按钮（第一个是"开始阅读"，指向第一篇文章）
indexbtn:
  enable: true
  name: Github
  icon: fab fa-github-alt
  url: https://github.com/Yang-li-young      # 改成你自己的 GitHub 主页

# banner 第二行小图标，留空即不显示
socialLink:
  github: https://github.com/Yang-li-young
  email: yourmail@example.com
  qq:
  weibo:
  zhihu:
  rss: true      # RSS 图标需装 hexo-generator-feed（见第六节）
```

### 4.4 首页特色模块（按需开/关）

| 配置段 | 作用 | 建议 |
|--------|------|------|
| `dream` | 首页"梦想"格言卡片 | 改成你喜欢的句子 |
| `music` | 首页吸底网易云音乐播放器 | `id` 换成你自己的歌单 ID（网易云网页版地址栏里 `playlist?id=` 后面那串数字）；不想要就 `enable: false` |
| `video` | 首页视频模块 | 默认关 |
| `recommend` | 推荐文章轮播 | 保持开，配合文章 front-matter `top: true` 使用 |
| `time` | 页脚"本站已运行 N 天" | 想要就 `enable: true` 并填你的建站日期 |
| `clicklove` / `sakura` / `mouseStar` / `snowdown` | 点击爱心 / 樱花 / 鼠标星星 / 飘雪特效 | **最多开一个**，多了卡 |

### 4.5 文章相关功能

```yaml
toc:
  enable: true              # 文章目录侧栏
  heading: h2, h3, h4

reward:                     # 文末打赏
  enable: true
  wechat: /medias/reward/wechat.png    # 放你自己的收款码到 source/medias/reward/
  alipay: /medias/reward/alipay.jpg

copyright:                  # 复制文章追加版权声明
  enable: true
  minCharNumber: 120

postInfo:                   # 需先安装 hexo-wordcount（见第六节）
  date: true
  update: true
  wordCount: true
  min2read: true

githubLink:                 # 页面右上角 "Fork me" 丝带
  enable: true
  url: https://github.com/Yang-li-young/Yang-li-young.github.io   # 改成你自己的仓库
```

### 4.6 "关于我"页面的个人信息

```yaml
profile:
  avatar: /medias/avatar.jpg      # 你的头像
  career: 学生 / 程序员 / …
  introduction: 你的自我介绍…

myProjects:        # 关于页"我的项目"卡片，改成你自己的仓库
  enable: true
  items:
    - name: 你的项目名
      url: https://github.com/you/repo
      description: 项目简介

mySkills:          # 技能进度条
  enable: true
  items:
    - name: JavaScript
      color: '#62b1dd'
      percentage: 80

myGallery:         # 关于页相册
  enable: false    # 不想放就关
```

### 4.7 评论系统（默认全关）

主题支持 Gitalk / Gitment / Valine / Waline / Twikoo / Disqus / Livere 等，**默认都不开启**，首页留言板（Contact 页）也依赖评论系统。选择建议：

| 方案 | 特点 | 适合 |
|------|------|------|
| **Twikoo**（推荐） | 免费可部署到 Vercel/腾讯云，无权限风险 | 想省事 |
| Waline | 需要自己部署服务端，功能全 | 想要完整控制 |
| Gitee/Gitalk | 基于 GitHub Issue，无需服务器 | **注意**：主题配置文件里的注释专门警告了 Gitalk OAuth App 权限过高（可读写授权者全部公共仓库），介意勿用 |
| 不开 | 纯静态展示 | 完全可以，很多人不开 |

以 Twikoo 为例（[官方快速开始](https://twikoo.js.org/)）：

```yaml
twikoo:
  enable: true
  envId: 你的环境ID     # 按 Twikoo 文档部署后获得
```

### 4.7.1 访问统计

```yaml
busuanziStatistics:       # 不蒜子访问量统计，无需注册
  enable: true
googleAnalytics:          # 有 GA 账号就填
  enable: false
  id:
```

## 五、创建功能页面

主题的 tags / categories / about 等页面需要**手动创建对应页面**才有内容。逐个执行：

```powershell
pnpm exec hexo new page "categories"
pnpm exec hexo new page "tags"
pnpm exec hexo new page "about"
pnpm exec hexo new page "contact"    # 可选：留言板
pnpm exec hexo new page "friends"    # 可选：友链
```

然后把生成的 `source/<名称>/index.md` 的 front-matter 改为对应类型（`date` 随意）：

```yaml
# source/categories/index.md
---
title: categories
date: 2026-09-17 00:00:00
type: "categories"
layout: "categories"
---
```

tags 页用 `type: "tags"` / `layout: "tags"`；about 页用 `type: "about"` / `layout: "about"`；contact 页用 `type: "contact"` / `layout: "contact"`。

### 5.1 友链数据文件（可选）

创建友链页后，还需要数据文件 `source/_data/friends.json`：

```json
[
  {
    "avatar": "https://example.com/avatar.jpg",
    "name": "朋友的名字",
    "introduction": "一句话介绍",
    "url": "https://example.com/",
    "title": "前去围观"
  }
]
```

### 5.2 404 页（可选）

直接新建 `source/404.md`：

```yaml
---
title: 404
type: "404"
layout: "404"
date: 2026-09-17 00:00:00
description: "Oops～，页面走丢了！"
---
```

## 六、安装推荐插件

```powershell
# 搜索（主题导航栏搜索框依赖）
pnpm add hexo-generator-search

# 文章字数统计 / 阅读时长（4.5 节 postInfo 依赖）
pnpm add hexo-wordcount

# 中文标题转拼音链接（强烈建议：避免链接里出现中文，利于 SEO 和分享）
pnpm add hexo-permalink-pinyin

# RSS 订阅（可选，socialLink.rss 需要它）
pnpm add hexo-generator-feed

# GitHub 风格 emoji，写 :smile: 会渲染成表情（可选）
pnpm add hexo-filter-github-emojis
```

安装后把以下配置追加到根目录 `_config.yml` 末尾：

```yaml
# 搜索
search:
  path: search.xml
  field: post

# 中文链接转拼音
permalink_pinyin:
  enable: true
  separator: '-'

# RSS
feed:
  type: atom
  path: atom.xml
  limit: 20
  order_by: -date

# emoji
githubEmojis:
  enable: true
  className: github-emoji
  inject: true
  styles:
  customEmojis:
```

装完插件后本地验证一遍：

```powershell
pnpm clean
pnpm server   # 检查：搜索框能搜到文章、字数统计出现、文章链接是拼音
```

## 七、写作与本地预览

### 7.1 新建文章

```powershell
pnpm exec hexo new "我的第一篇文章"        # 生成 source/_posts/我的第一篇文章.md
pnpm exec hexo new draft "还在酝酿的选题"   # 草稿，放 source/_drafts/，不会构建发布
pnpm exec hexo publish "还在酝酿的选题"     # 草稿定稿，移入 _posts
```

### 7.2 Front-matter 字段速查（Matery 专用）

文章头部的 YAML 区块，除 Hexo 标准字段外，本主题额外支持：

| 字段 | 默认值 | 作用 |
|------|--------|------|
| `top` | `false` | `true` 时置顶并进入首页"推荐"轮播 |
| `hide` | `false` | `true` 时不在首页列表显示 |
| `cover` | `false` | `true` 时加入首页封面轮播 |
| `coverImg` | 无 | 轮播图图片路径；不填则用文章特色图 |
| `img` | 主题随机图 | 文章列表卡片特色图，建议用图床 URL |
| `summary` | 自动截取 | 自定义卡片摘要 |
| `password` | 无 | 文章阅读密码（**SHA256 加密后的值**，且需主题 `verifyPassword.enable: true`） |
| `toc` | `true` | 单篇关闭目录 |
| `mathjax` | `false` | 单篇开启数学公式渲染（主题 `mathjax.enable: true` 前提下） |
| `reprintPolicy` | `cc_by` | 转载声明：`cc_by`/`cc0`/`noreprint`/`pay` 等 |
| `categories` / `tags` | 无 | 建议一篇文章**一个分类**多个标签 |

示例：

```yaml
---
title: 我的博客搭好了
date: 2026-09-17 22:00:00
categories: 建站
tags: [hexo, github-pages]
top: true
cover: true
coverImg: /medias/banner/0.jpg
summary: 记录从模板到上线的全过程
---
```

### 7.3 图片方案

- **推荐**：图床（[主题 README 同款建议](https://github.com/blinkfox/hexo-theme-matery)：腾讯云 COS、七牛云等）或仓库内路径，`img` 字段填 URL 或 `/medias/xxx.jpg`。
- **本地管理**：开了 `post_asset_folder: true` 后，`hexo new` 会自动建同名文件夹，图片放里面用相对路径引用。
- 注意：直接放 `source/` 根目录的图片会原样发布，建议统一放 `source/medias/`。

### 7.4 三个日常命令

```powershell
pnpm server    # 本地预览 http://localhost:4000（改配置/主题模板时大多会自动刷新）
pnpm build     # 生成静态文件到 public/，部署交给 Actions，一般不用手动执行
pnpm clean     # 清空缓存（db.json）和 public/——样式或文章显示不对时先 clean 再试
```

## 八、部署到 GitHub Pages（GitHub Actions 自动化）

你的 `main` 分支就是 GitHub Pages 用户站点的源码分支。**不要**把 `public/` 构建产物提交进 `main`（`.gitignore` 已忽略它），而是用 Actions 在云端构建发布：

1. 你的仓库存源码（`main`）
2. push 到 `main` → Actions 自动 `pnpm install` + `pnpm build`
3. 把 `public/` 作为 artifact 上传并发布到 Pages
4. 访问 `https://yang-li-young.github.io`

### 8.1 给 package.json 加包管理器声明

Actions 的 pnpm 装配步骤需要知道用哪个 pnpm。编辑 `package.json`，在 `"private": true,` 下面加一行：

```json
{
  "name": "hexo-site",
  "version": "00.0.0",
  "private": true,
  "packageManager": "pnpm@10.15.1",
  ...
}
```

### 8.2 创建工作流文件

新建 `.github/workflows/deploy.yml`（内容如下，可直接照抄）：

```yaml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch: {}   # 支持手动在 Actions 页面点 Run workflow

permissions:
  contents: read
  pages: write
  id-token: write

# 同一时间只允许一个部署，排队中重复触发的旧任务自动取消
concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v4   # 从 package.json 的 packageManager 字段读版本

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build site
        run: pnpm build               # 即 hexo generate

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 8.3 打开仓库的 Pages 设置

GitHub 网页 → 你的 `Yang-li-young.github.io` 仓库 → **Settings** → **Pages** → Build and deployment 的 **Source** 选择 **GitHub Actions**（而不是 "Deploy from a branch"）。

### 8.4 首次部署

把目前为止的所有改动提交并推到 `main`：

```powershell
git add -A
git commit -m "切换 matery 主题并接入自动部署"
git push origin main    # 或走 dev 分支 PR 合并（见第九节）
```

推完后到仓库 **Actions** 标签页看构建进度（首次约 1~2 分钟），绿勾后访问：

**https://yang-li-young.github.io**

> 失败了怎么办：点进失败的 run 看日志。最常见三种：① 主题目录是空的（没删 `.git`，重做 2.1）；② `pnpm install --frozen-lockfile` 报锁文件不一致（本地跑过 `pnpm add` 后忘了提交 lockfile）；③ YAML 缩进错误。

### 8.5 绑定自定义域名（可选）

1. 域名服务商加解析记录：`CNAME` 或 `A` 记录指向 GitHub Pages（规则见 [官方文档](https://docs.github.com/pages)）
2. 仓库 Settings → Pages → Custom domain 填入域名并勾选 **Enforce HTTPS**
3. 在 `source/` 下建一个只含域名的 `CNAME` 文件：`echo yourdomain.com > source/CNAME`（不加的话每次构建都会覆盖 Pages 的域名配置）
4. 根目录 `_config.yml` 的 `url` 同步改成 `https://yourdomain.com`

## 九、日常发布流程（配合你的分支模型）

你已经有 `dev` → PR → `main` 的分支习惯（main 上已合并过 PR #1），建议日常这样发布：

1. **写文章**：在 `dev` 分支（或每篇文章单独开个分支）写、改、预览
2. **提 PR** → 自己审一遍 diff → 合并进 `main`
3. **自动上线**：合并即触发 8.2 的工作流，1~2 分钟后线上生效

懒得开 PR 的时候直接 push 到 `main` 也一样会触发部署；改了主题模板或配置拿不准效果时，也可以在 Actions 页面手动 **Run workflow** 重跑一次。

> Dependabot 每天会自动开依赖升级 PR（`.github/dependabot.yml` 配的），合并它们即可让依赖保持更新；Actions 会在云端验证升级后能否正常构建。

## 十、常见问题（FAQ）

| 症状 | 处理 |
|------|------|
| 改了配置/文章但预览没变化 | `pnpm clean` 后重来；`db.json` 缓存偶尔会脏 |
| 代码块里出现 `&#123;` `&#125;` | 是装了 `hexo-prism-plugin` 的症状，本仓库没装；若将来装了请 `pnpm remove hexo-prism-plugin` |
| 文章链接带中文且很长 | 确认 `permalink_pinyin.enable: true` 已装插件并写进根 `_config.yml` |
| 本地正常、线上样式全乱 | ① 根 `_config.yml` 的 `url` 没改对；② 构建产物过期，重跑 Actions |
| Actions 里主题目录是空的 | 没做 2.1（删主题内嵌 `.git`），或主题文件夹没提交进仓库 |
| 首页 banner 是别人的图 | 没放 `source/medias/banner/0~6.jpg` 覆盖 |
| 评论框不出现 | 对应评论系统 `enable` 没开，或（Contact 页）该评论系统未配置 |
| 想隐藏某篇不想发的文章 | front-matter 加 `hide: true`，或移入 `source/_drafts/` |

## 十一、上线检查清单

- [ ] `themes/hexo-theme-matery/.git` 已删除，主题目录已提交进仓库
- [ ] `hexo-theme-landscape` 依赖已移除，`theme: hexo-theme-matery`
- [ ] `_config.hexo-theme-matery.yml` 覆盖文件已建好，个性化改在这份里
- [ ] 根 `_config.yml`：站点信息、`url`、`language: zh-CN`、`syntax_highlighter: prismjs`、`per_page: 12`
- [ ] `source/favicon.png`、`source/medias/logo.png`、`avatar.jpg`、`banner/0~6.jpg`、打赏码等已就位
- [ ] categories / tags / about（及可选的 contact、friends、404）页面已创建
- [ ] 插件已装：search、wordcount、permalink-pinyin（+ 可选 feed、emojis）
- [ ] `package.json` 已加 `"packageManager": "pnpm@10.15.1"`
- [ ] `.github/workflows/deploy.yml` 已提交，Settings → Pages → Source = GitHub Actions
- [ ] `pnpm clean && pnpm server` 本地全流程跑通，首页、文章页、404 都正常
- [ ] `README.md` 里"当前使用 Landscape 主题"的描述已同步更新（第 115 行附近）
