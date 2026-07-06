# Wu Portfolio

这是一个基于 HugoBlox 的静态个人作品集网站。项目内容主要由 Markdown 和 YAML 管理，构建后输出为纯静态文件，可部署到 GitHub Pages、Netlify、Vercel 或 Cloudflare Pages。

## 项目定位

- 个人作品集首页
- 项目展示
- 技术栈展示
- 工作经历时间线
- 技术博客
- 联系方式与简历下载
- 静态搜索索引

## 核心框架

| 层级 | 技术 |
| :--- | :--- |
| 静态站点生成器 | Hugo Extended |
| 站点框架 | HugoBlox |
| 模块系统 | Hugo Modules / Go Modules |
| 内容格式 | Markdown / YAML |
| 样式系统 | Tailwind CSS v4 |
| 搜索索引 | Pagefind |
| 前端运行时 | Preact |
| 包管理 | pnpm via Corepack |
| 部署 | GitHub Pages / Netlify 配置均存在 |

## 技术栈说明

### Hugo

Hugo 负责把 `content/`、`data/`、`config/`、`assets/` 和 `static/` 中的内容构建成静态网站。

当前项目使用 Hugo Extended，版本由 `hugoblox.yaml` 指定：

```yaml
build:
  hugo_version: '0.162.0'
```

本地实际验证过的 Hugo 版本：

```text
hugo v0.163.3+extended+withdeploy
```

### HugoBlox

HugoBlox 提供页面区块、作品集模板、主题系统和组件能力。项目通过 Hugo Modules 引入 HugoBlox 模块：

```yaml
imports:
  - path: github.com/HugoBlox/kit/modules/integrations/netlify
  - path: github.com/HugoBlox/kit/modules/blox
  - path: github.com/HugoBlox/kit/modules/slides
```

相关配置文件：

- `hugoblox.yaml`
- `config/_default/module.yaml`
- `config/_default/params.yaml`

### Tailwind CSS

项目依赖 Tailwind CSS v4 和 `@tailwindcss/cli`，主要用于 HugoBlox 构建过程中的样式处理。

相关依赖位于 `package.json`：

```json
{
  "@tailwindcss/cli": "^4.1.12",
  "@tailwindcss/typography": "^0.5.10",
  "tailwindcss": "^4.1.12"
}
```

### Pagefind

Pagefind 用于为静态站点生成搜索索引。构建完成后会在 `public/pagefind/` 下生成搜索索引文件。

构建脚本：

```json
"build": "hugo --minify && pagefind --site public"
```

## 目录结构

```text
.
├── assets/                  # Hugo 资源文件，例如头像、图标、自定义资源
├── config/_default/          # Hugo 和 HugoBlox 配置
│   ├── hugo.yaml             # Hugo 基础配置，例如 baseURL
│   ├── languages.yaml        # 多语言配置
│   ├── menus.yaml            # 顶部导航菜单
│   ├── module.yaml           # Hugo Modules 配置
│   └── params.yaml           # HugoBlox 站点参数、主题、SEO、功能开关
├── content/                  # 页面和文章内容
│   ├── _index.md             # 首页 landing page 配置
│   ├── blog/                 # 博客文章
│   └── projects/             # 项目展示内容
├── data/authors/             # 作者资料、技能、经历、社交链接
├── static/                   # 原样复制到 public 的静态文件
│   └── uploads/resume.pdf    # 简历文件
├── .github/workflows/        # GitHub Actions 构建与部署
├── go.mod                    # Hugo Modules / Go Modules 依赖
├── package.json              # Node 依赖与构建脚本
├── pnpm-lock.yaml            # pnpm 锁文件
├── netlify.toml              # Netlify 构建配置
└── hugoblox.yaml             # HugoBlox 项目配置
```

## 关键内容文件

| 文件 | 作用 |
| :--- | :--- |
| `content/_index.md` | 首页结构，包含 hero、projects、skills、experience、blog、contact、CTA 等区块 |
| `data/authors/me.yaml` | 站点所有者信息，包括姓名、角色、邮箱、社交链接、教育、经历、技能 |
| `content/projects/*/index.md` | 项目详情页，每个目录对应一个项目 |
| `content/blog/*/index.md` | 博客文章，每个目录对应一篇文章 |
| `config/_default/hugo.yaml` | Hugo 基础配置，尤其是 `baseURL` |
| `config/_default/menus.yaml` | 导航菜单配置 |
| `config/_default/params.yaml` | 主题、SEO、搜索、评论、分析、页脚等全局配置 |

## 首页区块

首页由 `content/_index.md` 中的 `sections` 定义，当前包含：

| 区块 ID | Block | 作用 |
| :--- | :--- | :--- |
| `hero` | `dev-hero` | 首页主视觉、头像、身份介绍、打字机文案、CTA |
| `projects` | `portfolio` | 项目作品集，支持按标签过滤 |
| `skills` | `tech-stack` | 技术栈图标展示 |
| `experience` | `resume-experience` | 工作经历时间线 |
| `blog` | `collection` | 最近博客文章列表 |
| `contact` | `contact-info` | 联系方式 |
| 无 ID | `cta-card` | 简历下载和机会说明 |

## 本地环境要求

需要安装：

- Go
- Hugo Extended
- Node.js 22+
- Corepack

当前项目声明使用 pnpm：

```json
"packageManager": "pnpm@10.14.0"
```

如果本机不能直接使用 `pnpm` 命令，可以使用 Corepack：

```bash
corepack pnpm --version
```

## 本地开发

### 首次运行

如果 Go 命令在 `~/.bash_profile` 中配置，需要先加载：

```bash
source ~/.bash_profile
```

确认基础命令可用：

```bash
go version
hugo version
node --version
corepack pnpm --version
```

安装 Node 依赖：

```bash
corepack pnpm install
```

### 开发模式运行

启动 Hugo 本地开发服务器：

```bash
source ~/.bash_profile
HUGO_CACHEDIR="$PWD/.hugo_cache" corepack pnpm run dev
```

默认访问地址通常是：

```text
http://localhost:1313
```

开发模式会监听内容和配置变化，修改 Markdown、YAML 或资源文件后会自动刷新页面。

### 生产构建

生成生产环境静态文件：

```bash
source ~/.bash_profile
HUGO_CACHEDIR="$PWD/.hugo_cache" corepack pnpm run build
```

构建输出目录：

```text
public/
```

构建流程：

1. Hugo 生成静态网站到 `public/`
2. Pagefind 扫描 `public/`
3. Pagefind 生成搜索索引到 `public/pagefind/`

### 本地预览生产构建

构建完成后，如果只想预览 `public/` 中的静态产物，可以运行：

```bash
npx serve public
```

或者使用任意静态文件服务器，例如 Python：

```bash
cd public
python3 -m http.server 8080
```

然后访问：

```text
http://localhost:8080
```

## 构建

```bash
source ~/.bash_profile
HUGO_CACHEDIR="$PWD/.hugo_cache" corepack pnpm run build
```

构建输出目录：

```text
public/
```

构建流程：

1. Hugo 生成静态网站到 `public/`
2. Pagefind 扫描 `public/`
3. Pagefind 生成搜索索引到 `public/pagefind/`

最近一次本地验证结果：

```text
Pages: 64
Static files: 1
Processed images: 95
Pagefind indexed pages: 6
Pagefind indexed words: 1506
```

## 部署

### GitHub Pages 自动部署

项目已经包含 GitHub Pages 自动部署工作流：

```text
.github/workflows/deploy.yml
.github/workflows/build.yml
```

当前 `hugoblox.yaml` 指定部署目标为 GitHub Pages：

```yaml
deploy:
  host: 'github-pages'
```

自动部署触发方式：

1. 推送到 `main` 分支时自动触发
2. 在 GitHub Actions 页面手动触发 `Deploy website to GitHub Pages` workflow

部署流程：

1. `.github/workflows/deploy.yml` 读取 `hugoblox.yaml`，确认部署目标是否为 `github-pages`
2. 调用 `.github/workflows/build.yml` 执行构建
3. 安装 Node、pnpm、Go、Hugo Extended
4. 运行 Hugo 生成 `public/`
5. 运行 Pagefind 生成搜索索引
6. 上传 `public/` 为 GitHub Pages artifact
7. 发布到 GitHub Pages

GitHub 仓库需要开启 Pages：

1. 进入仓库 `Settings`
2. 打开 `Pages`
3. `Build and deployment` 选择 `GitHub Actions`
4. 推送代码到 `main` 分支
5. 在 `Actions` 中查看部署状态

注意：当前 workflow 只在 `main` 分支 push 时自动部署。如果日常开发使用 `develop` 分支，需要合并到 `main` 后才会自动发布。

### 手动发布流程

本地确认构建通过：

```bash
source ~/.bash_profile
HUGO_CACHEDIR="$PWD/.hugo_cache" corepack pnpm run build
```

提交并推送到 `main`：

```bash
git add .
git commit -m "docs: update local run and deployment guide"
git push origin main
```

如果当前在 `develop` 分支，常见流程是：

```bash
git checkout main
git merge develop
git push origin main
```

### Netlify

项目同时保留了 Netlify 配置：

```text
netlify.toml
```

注意：当前仓库同时存在 GitHub Pages 和 Netlify 配置。建议实际部署时选择一个主部署平台，避免配置长期不一致。

## 常见问题

### `go: binary with name "go" not found in PATH`

先加载本地环境配置：

```bash
source ~/.bash_profile
```

然后确认：

```bash
go version
```

### `pnpm: command not found`

不要依赖全局 pnpm，可直接使用 Corepack：

```bash
corepack pnpm install
corepack pnpm run build
```

### `corepack enable` 权限错误

如果出现类似错误：

```text
EACCES: permission denied, symlink ... -> /usr/local/bin/pnpm
```

说明当前用户没有 `/usr/local/bin` 写权限。可以跳过 `corepack enable`，直接使用：

```bash
corepack pnpm <command>
```

### Hugo cache 权限错误

如果 Hugo 试图写入系统缓存目录失败，可以把缓存放到项目目录：

```bash
HUGO_CACHEDIR="$PWD/.hugo_cache" corepack pnpm run build
```

## 当前注意事项

- `config/_default/hugo.yaml` 中的 `baseURL` 仍需要按最终域名修改。
- `data/authors/me.yaml` 和 `content/_index.md` 中仍可能包含模板示例个人信息，需要替换为真实内容。
- 推荐统一使用 `pnpm-lock.yaml`，不要混用 `package-lock.json`。
- `.hugo_cache/`、`public/`、`resources/` 属于构建或缓存产物，不应提交到仓库。
