# Local Development

本地预览和开发这个 Jekyll site 的步骤。

## 一次性环境配置

### 1. 安装 Ruby（用 rbenv 管理版本）

```bash
brew install rbenv ruby-build

# 把 rbenv 初始化加到 shell config
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
source ~/.zshrc

# 装稳定版本
rbenv install 3.3.0
rbenv global 3.3.0

# 验证
ruby -v   # 应该显示 ruby 3.3.0
```

如果 `ruby -v` 仍然显示系统自带的 2.6.10：

- 检查 rbenv 初始化：`cat ~/.zshrc | grep rbenv`
- 检查 PATH：`echo $PATH`，应该看到 `~/.rbenv/shims` 在前面
- 检查实际用的 ruby：`which ruby`，应该是 `~/.rbenv/shims/ruby`

### 2. 安装 Jekyll 和 Bundler

```bash
gem install jekyll bundler
```

### 3. 进入 repo 目录，安装依赖

```bash
cd lecaibio.github.io
bundle install
```

第一次会下载一堆 dependencies，1-3 分钟。

## 日常使用

### 启动本地 server

```bash
cd lecaibio.github.io
bundle exec jekyll serve
```

打开浏览器访问 `http://localhost:4000`。

Live reload 自动启用——改 `index.md` 或新加 post 文件，保存后浏览器自动刷新。

按 Ctrl+C 停止 server。

### 添加新 post

在 `_posts/` 文件夹下创建文件，文件名格式严格遵守：

```
YYYY-MM-DD-title-with-hyphens.md
```

例：`_posts/2026-05-15-clinical-ml-real-data.md`

文件内容开头必须有 front matter：

```markdown
---
title: "文章标题（可中英文）"
date: 2026-05-15
layout: post
---

正文从这里开始...
```

保存后自动出现在首页 Writing 列表。

### 添加图片

放在 repo 根目录或建 `assets/` 文件夹。在 markdown 里引用：

```markdown
![描述](/assets/images/filename.png)
```

一篇文章的图放同一个子目录，用文章的短名，例如
`/assets/images/clinical_field_notes/01-three-rates.png`。SVG 跟 PNG 一样用 `![]()` 引用，
Jekyll 不需要额外配置。

### 文末 note 格式

每篇文章结尾用同一个块。**照抄这个结构**，不要换成普通段落或 `>` 引用：

```markdown
<br><br>

---

<small>
**A note on this piece**
<br>
第一段。
<br>
第二段。
</small>
```

要点：

- `<br><br>` 空开，然后 `---` 分隔线，然后整块包在 `<small>` 里
- 标题固定是 **A note on this piece**，加粗，单独一行
- 段落之间用单个 `<br>`，不要空行分段（`<small>` 块里 markdown 的段落解析不可靠）
- **块内的链接要用 `<a href="...">` HTML 写法**，不要用 `[]()`。kramdown 在 `<small>` 里
  不保证解析 markdown 链接

这个块放什么：致谢、数据和代码出处、方法上的免责（比如"未加权"、"这个结论是推理不是实验"）、
AI 协助声明、以及跟雇主无关的声明。正文里不写这些，全部收到文末。

## Repo 结构

```
lecaibio.github.io/
├── _config.yml          # Jekyll 配置（title, theme, repository）
├── _posts/              # 所有 blog 文章
├── assets/              # 图片等静态资源（按需创建）
├── index.md             # 首页 about + posts 列表
├── Gemfile              # Ruby 依赖声明
├── Gemfile.lock         # 锁定的依赖版本（自动生成）
├── DEVELOPMENT.md       # 本文件
└── _site/               # Jekyll build 输出（git ignore）
```

## 常见 gotchas

- **改了 `_config.yml` 没生效**：需要重启 server。Ctrl+C 然后 `bundle exec jekyll serve`。其他文件改动不需要重启。

- **端口 4000 被占用**：`bundle exec jekyll serve --port 4001`

- **`webrick` 报错（Ruby 3.x）**：在 Gemfile 加 `gem "webrick"`，再 `bundle install`。

- **GitHub Metadata 报 "No repo name found"**：`_config.yml` 里要有 `repository: lecaibio/lecaibio.github.io`。

- **post 不显示**：检查文件名格式（必须 `YYYY-MM-DD-`）、front matter（必须有 `layout: post`）、文件位置（必须在 `_posts/` 里）。

- **本地预览跟线上不一致**：`github-pages` gem 锁定了 GitHub 实际使用的 Jekyll 版本，本地用它就跟线上一致。如果偏差很大，`bundle update` 更新依赖。

- **换了 theme 之后 favicon 没了**：`_includes/head.html` 是本地覆盖 minima 2.5.1 的同名文件，
  favicon 的 `<link>` 就加在里面（2.5.1 没有 `custom-head.html` 这个 hook，3.x 才有）。
  升级 theme 要重新从 gem 里 copy 一份 head.html 再把那几行加回去。文件里有注释提醒。

- **favicon 文件放在哪**：

  ```
  favicon.ico                    # 必须在根目录：浏览器会盲请求 /favicon.ico，不读 HTML
  assets/
    favicon.svg                  # 源文件；下面三个都是从它光栅化出来的
    favicon-32.png               # 这四个由 _includes/head.html 的 <link> 显式指名
    favicon-16.png
    apple-touch-icon.png         # 180px，无圆角（iOS 自己切）
  ```

  只有根目录那份 `.ico` 会被盲请求，别在 `assets/` 下再放一份。

- **本地 `assets/` 会不会盖掉 theme 的？不会，是合并**。minima 自己也有 `assets/`，build 之后
  `_site/assets/` 里 `main.css`、`minima-social-icons.svg` 和你自己的文件是并存的。放心用。

## 不本地预览的备选

如果只是改 typo 或小段落，直接 push 到 GitHub，等 1-2 分钟刷新 `https://lecaibio.github.io` 也能看到效果。本地预览主要价值是写长文章时反复调整格式。

## SSH 双账户提醒

这个 repo 必须用 lecaibio 账户的 SSH 别名。Clone 时用：

```bash
git clone git@github.com-lecaibio:lecaibio/lecaibio.github.io.git
```

进入 repo 后第一次设置：

```bash
git config user.name "Le Cai"
git config user.email "lecai@alumni.stanford.edu"
```

详细参考 `~/LeCaiBio/reminder-to-gitclone.md`。

---

最后更新：2026-08-15
