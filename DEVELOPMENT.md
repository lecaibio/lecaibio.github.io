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
description: "一到两句话说清楚这篇的结论。"
date: 2026-05-15
layout: post
---

正文从这里开始...
```

保存后自动出现在首页 Posts 列表。

**`description` 一个字段两个用途**：首页列表里显示在标题下面，同时被 jekyll-seo-tag 拿去做
`<meta name="description">`（也就是 Google 搜索结果里标题下面那行）。不写的话 seo-tag 会退回去
用标题本身，等于标题重复一遍，浪费。

写法要点：

- **写结论，不写题材**。"Synthea 的变量是通过 care protocol 相关的，不是通过 biology" 比
  "关于合成临床数据的一些思考" 有用得多
- 控制在 **150–165 个字符**，再长 Google 会截断
- 正文第一段不要跟它逐字重复。首页读到 description、点进来又读一遍同一句话，很没意思

**正文不要再写一个 `# 标题`**。`layout: post` 已经把 front matter 的 `title` 渲染成 `<h1>` 了，
正文再写一遍会出现两个 h1，对 SEO 和读者都是重复。正文直接从第一段开始，小标题从 `##` 起。

### 添加图片

放在 repo 根目录或建 `assets/` 文件夹。在 markdown 里引用：

```markdown
![描述](/assets/images/filename.png)
```

一篇文章的图放同一个子目录，用文章的短名，例如
`/assets/images/clinical_field_notes/01-three-rates.png`。SVG 跟 PNG 一样用 `![]()` 引用，
Jekyll 不需要额外配置。

### 打 tag

front matter 里加一行，两三个就够：

```yaml
tags: [clinical ml, synthetic data]
```

现在在用的：

| tag | 意思 |
|---|---|
| `clinical ml` | 真实病人数据上的建模 |
| `synthetic data` | 合成数据、生成器、transfer |
| `bio + ai` | 行业观察、方向判断 |
| `career` | 路径、学习、转行 |
| `infrastructure` | 底下那层：生产运维（AWS）和分析工作的底层（git、CI）都算 |
| `llm` | 大模型怎么用：文本管线、分类、成本和数据边界 |
| `reproducibility` | 环境、seed、记录、别人能不能把这个分析跑第二遍 |
| `side projects` | 自己维护的东西 |
| `industry` | 药企、会议、市场 |
| `中文` | 中文写的文章 |

**tag 的价值在复用**。同一个 tag 至少要能盖住两篇文章，只出现一次的 tag 等于没有分类，
只是给标题加了个副标题。想加新 tag 之前先看看上面这张表能不能塞进去。

`/tags/` 页面（`tags.html`）是从 `site.tags` 自动生成的，加了新 tag 不用改任何模板，
build 一次就出现了。这个页面用纯 Liquid 写的，没有插件，所以 GitHub Pages 能跑
（`jekyll-archives` 那类插件不行，GitHub Pages 不允许）。

**非 ASCII 的 tag（比如 `中文`）要注意**：模板里用的是 `slugify: 'raw'`，只把空格换成
连字符，其他字符原样保留。默认的 `slugify` 会把中文整个吃掉变成空字符串，锚点就断了。
`_includes/tag-list.html` 和 `tags.html` 两边必须用同一个 mode，否则链接对不上。

### 字数和阅读时间

"1727 words · 8 min read" 两个地方都有：首页放在 description 下面，文章里放在标题的
日期那一行下面。字号比日期和 tag 小一号、颜色更浅——日期和 tag 是用来找文章的，字数不是。
build 的时候自动算，**不用在 front matter 里写任何东西**。逻辑在
`_includes/reading-time.html`。

中英文分开算，因为 GitHub Pages 锁的 Jekyll 3.10 里 `number_of_words` 就是一句
`input.split.length`——按空格切。中文没有空格，那篇中文文章 1723 个字会被数成 109 个
"word"，直接按英文算就变成 "1 min read" 了。（Jekyll 4 才有 CJK 模式，用不了。）

所以是按**每个词平均多少字符**来判断语种的：

| | 每词字符数 |
|---|---|
| 三篇英文 | 4.7 / 4.8 / 5.0 |
| 一篇中文 | 15.8 |

阈值取 9，卡在中间。中文文章里夹不少英文术语和代码也掉不到 9 以下，英文文章也不可能
高到 9 以上。速度用的是英文 220 词/分钟、中文 400 字/分钟，都向上取整，所以再短的文章
也是 "1 min" 不会是 "0 min"。

真要覆盖（比如某篇判断错了），改 `_includes/reading-time.html` 里那个 `ratio > 9`。

### 文章目录（Contents）

每篇文章标题下面那个 Contents 方框，是 build 的时候从正文的 `##` 标题自动生成的，
**不用写任何东西**。逻辑在 `_includes/toc.html`。

规则：

- `##` 一定进目录；`###` 只有在**这篇 `##` 不超过 5 个**的时候才嵌进去显示。已经有很多
  `##` 的文章不需要第二层就能找到东西，硬加反而更难读——学习路径那篇 9 个 `##` 加 17 个
  `###`，全列出来 26 行，比它指向的一些小节还长
- **总条目少于 4 条的文章不显示**。两三行的目录是装饰不是导航
- 锚点用的是 kramdown 自动生成的 heading id，中文标题也能用（`#两条赛道各有千秋`）
- 桌面两栏，手机一栏，切换在 `main.scss` 的 `.post-toc-list` 里

**写长文章的时候可以利用这个结构**。Synthea 那篇原来是平铺的 9 个 `##`，1700 字分成 9 段，
平均一段 190 字，目录看上去就是一串没有主次的标题。后来按论证的三段式重新分组：先摆事实
（比较），再讲原因（读代码），最后讲影响（对 ML 意味着什么），加一个结尾。变成 4 个 `##`
带 8 个 `###`，目录一眼能看出文章的骨架。**先想清楚文章分几个大块，再写小标题**，比写完
一堆小标题再补结构容易。

所以**正文小标题一律用 `##`**，不要用 `#`。用 `#` 有两个后果：页面上出现多个 h1
（`layout: post` 已经拿 front matter 的 title 渲染了一个），而且目录抓不到它。

改阈值（4 个标题）在 `_includes/toc.html` 里的 `heading_count >= 4`。

### 正文里的强调格式

两个自定义样式，都定义在 `assets/main.scss` 里。

**高亮一句话**，直接用 `<mark>`，句子留在段落里不要单拎出来：

```markdown
它把 protocol 编码进去了。<mark>module library 才是 Synthea 的产品，合成病人只是它的
输出。</mark>大部分人只拿输出，不打开 module。
```

**把一段话framed起来**用 `.callout`，注意 `markdown="1"`，否则块里的 markdown 不解析：

```markdown
<div class="callout" markdown="1">
这里是一段需要跟上下文分开的话，两到四句为宜。
</div>
```

两个都克制用。一篇文章里 `<mark>` 一处、`.callout` 一处，超过就没有强调效果了。

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
├── _config.yml          # Jekyll 配置（title, description, theme, repository）
├── _posts/              # 所有 blog 文章
├── _includes/
│   ├── head.html        # 覆盖 minima：加 favicon
│   ├── footer.html      # 覆盖 minima：一行，名字 + 图标链接
│   ├── tag-list.html    # 自己写的：渲染一篇文章的 tag
│   ├── reading-time.html # 自己写的：算字数和阅读时间
│   └── toc.html         # 自己写的：从 h2 生成文章目录
├── _layouts/
│   ├── home.html        # 覆盖 minima：列表加 description 和 tag
│   └── post.html        # 覆盖 minima：meta 行加 tag
├── _sass/
│   ├── _variables.scss  # 改 minima 的变量（字号、行高、链接色）
│   └── _custom.scss     # 本站自己加的样式，改样式基本都在这
├── assets/
│   ├── main.scss        # 三行的壳，只负责按顺序 import 上面两个和 minima
│   ├── images/          # 图片；about/ 放头像，其余按文章分子目录
│   └── favicon*         # 见下面 favicon 那条
├── index.md             # 首页 about + posts 列表
├── tags.html            # /tags/ 页面，从 site.tags 自动生成
├── Gemfile              # Ruby 依赖声明
├── Gemfile.lock         # 锁定的依赖版本（自动生成）
├── DEVELOPMENT.md       # 本文件
└── _site/               # Jekyll build 输出（git ignore）
```

**一共覆盖了 minima 五个文件**：`_includes/head.html`、`_includes/footer.html`、
`_layouts/home.html`、`_layouts/post.html`、`assets/main.scss`。升级 theme 的时候这五个都要
重新从 gem 里 copy 一份再把改动加回去，每个文件开头都有 comment 说明改了什么。
`_includes/tag-list.html` 和 `tags.html` 是自己写的，minima 里没有同名文件，不用管。

gem 的位置：

```bash
bundle show minima
```

**导航栏（右上角，手机上是汉堡菜单）列的是「有 `title` 的 page」**，不是 post。
`index.md` 故意没有 title，所以不进导航；`tags.html` 有，所以导航里现在是一个 "Tags"。
一个有 title 的 page 都没有的时候，minima 照样会画出汉堡按钮，但点开是空的——之前 iPhone
上点了没反应就是这个原因。以后再加 page（比如 `about.html`）想让它进导航就给个 `title`。

## 常见 gotchas

- **改了 `_config.yml` 没生效**：需要重启 server。Ctrl+C 然后 `bundle exec jekyll serve`。其他文件改动不需要重启。

- **端口 4000 被占用**（`Address already in use - bind(2) for 127.0.0.1:4000`）：一般是上一个
  jekyll serve 没退干净。先看看是谁占着：

  ```bash
  lsof -nP -iTCP:4000 -sTCP:LISTEN
  ```

  确认是自己的 ruby/jekyll 进程之后 `kill <PID>`。4000 是个很常用的默认端口，不一定是
  Jekyll，所以先看再 kill。懒得管的话直接换端口：`bundle exec jekyll serve --port 4001`。

- **`webrick` 报错（Ruby 3.x）**：在 Gemfile 加 `gem "webrick"`，再 `bundle install`。

- **GitHub Metadata 报 "No repo name found"**：`_config.yml` 里要有 `repository: lecaibio/lecaibio.github.io`。

- **post 不显示**：检查文件名格式（必须 `YYYY-MM-DD-`）、front matter（必须有 `layout: post`）、文件位置（必须在 `_posts/` 里）。

- **改文件名 = 改 URL = 断链**。URL 是从文件名（不是从 `title`）生成的，
  `2026-08-15-foo.md` → `/2026/08/15/foo.html`。文章一旦发出去过，重命名文件（或者改
  title 之后顺手改文件名）会让旧链接 404。两个办法：

  - 还没人拿到链接 → 现在改，越早越好
  - 已经发出去了 → 在 front matter 里写死旧路径：

    ```yaml
    permalink: /2026/08/15/旧的文件名.html
    ```

  `2026-08-15-where-synthea-gets-its-correlations-phq9.md` 就是后一种情况：文件名和
  URL 对不上是故意的，动 `permalink` 那行之前先确认没人在用旧链接。

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

- **换首页照片**：`assets/images/about/le-cai.jpg`，600×867，就是原图的完整构图，只缩小没有
  裁。显示尺寸桌面 224px 宽、手机 190px 宽，都在 `main.scss` 的 `.intro-photo` 里，高度是
  `auto`，所以换成别的比例也不会变形。原图留在同目录下的 `Le_Cai_life.jpg`。

  **别直接把手机原图放进去**——原图 2.5MB，缩到 600px 宽是 239KB，小 90%，肉眼没差别。
  `-Z` 是按长边缩放，比例自动保持：

  ```bash
  cd assets/images/about
  sips -Z 867 Le_Cai_life.jpg --out le-cai.jpg          # 长边缩到 867
  sips -s formatOptions 72 le-cai.jpg --out le-cai.jpg  # JPEG 质量 72
  ```

  换完记得改 `index.md` 里 `<img>` 的 `width` / `height`。那两个属性不控制显示大小（CSS 才
  控制），但浏览器靠它们在图片加载完之前把位置留出来，写错了页面会在加载时跳一下。

- **`_config.yml` 里的 `description`**：不写的话 jekyll-seo-tag 会去拿 GitHub repo 的
  description（因为配了 `repository:`，jekyll-github-metadata 会注入）。那个字段在 GitHub
  仓库设置里改，不在这个 repo 里，容易忘。所以显式写在 `_config.yml` 里。

- **`Gemfile.lock` 现在是 ignore 掉的**。GitHub Pages build 的时候根本不看这个文件，它用
  自己那套固定版本，所以 commit 与否**不影响线上**，只影响本地。

  - 不 commit（现在这样）：每台机器 `bundle install` 各装各的，`bundle update` 不会产生
    需要处理的 diff
  - commit（更常见的做法）：换台机器 clone 下来装到的版本跟这台完全一样，本地预览的结果
    可复现。这个 repo 会在多台机器上 clone，所以其实更推荐这个

  想换成 commit：把 `.gitignore` 里 `Gemfile.lock` 那行删掉，然后
  `git add -f Gemfile.lock`。

- **`.claude/` 和 `.vscode/` 是故意 commit 的，不要 ignore**。里面是这个项目的配置，不是
  临时文件：`.vscode/settings.json` 管住了 `main.scss` 的假报错，`.claude/launch.json` 是
  本地起 server 的配置。换台机器 clone 下来就能直接用。只有 `settings.local.json` 那种
  带个人设置的才 ignore 掉，规则已经写在 `.gitignore` 里了。

- **VSCode 说 `assets/main.scss` 开头有错，但 `jekyll serve` 一切正常**：不是真错误。
  Jekyll 要求这个文件以 YAML front matter（`---` 那两行）开头，否则不会编译它、直接原样
  copy 过去。而 `---` 不是合法 SCSS，编辑器按纯 SCSS 解析就会报错。Jekyll 在交给 Sass
  之前会把 front matter 剥掉，所以编译没问题。

  已经处理了：真正的样式都挪到 `_sass/` 里（那两个文件没有 front matter，VSCode 能正常
  检查），`assets/main.scss` 只剩三行 import。`.vscode/settings.json` 里把这一个文件
  associate 成 plaintext，红线就没了，`_sass/` 的检查照常。

- **`_sass/` 里的文件写了中文或者破折号就编译失败**（`Invalid US-ASCII character "\xE2"`）：
  带 front matter 的文件是 Jekyll 按 UTF-8 读的，`_sass/` 里的 partial 是 Sass 自己读的，
  默认按 US-ASCII。所以 **`_sass/` 下每个文件第一行都要有**：

  ```scss
  @charset "utf-8";
  ```

  minima 自己的 `minima.scss` 第一行就是这个，同样的原因。注释里一个中文字、一个破折号
  （`—`）就足够让整个 build 挂掉。

- **footer 的图标**：来自 theme 自带的 `assets/minima-social-icons.svg`（github / linkedin /
  rss / twitter / mastodon 等十几个都在里面），不用自己找图标。链接地址读的是 `_config.yml`
  里的 `email` / `github_username` / `linkedin_username`，改链接改那里，不用动模板。

  两个坑：

  - sprite 里的 `<symbol>` **没有 viewBox**，所以图标只能按原始的 16×16 显示，
    在 CSS 里改 `.svg-icon` 的宽高只会把图标裁掉，不会缩放。要换尺寸得先给 symbol 补 viewBox
  - sprite 里**没有 email 图标**，信封是直接内联在 `footer.html` 里的，同样画在 16×16 上

  信箱那个 `<a>` 只有图标没有文字，所以每个链接都要 `aria-label`，否则读屏软件读不出来。

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
