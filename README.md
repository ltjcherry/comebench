# CoMeBench — project page

A static, **content-driven** site for the CoMeBench dataset. Text lives in
`content.json`, the page renders itself from it, and editors change content
through a web admin (Pages CMS) — no code required.

## 结构

```
CoMeBench-site/
├─ index.html      ← 空壳：只放 <head> 和容器，不写任何文案
├─ content.json    ← 全部内容（人写的都在这）
├─ render.js       ← 渲染器：读 content.json，生成页面
├─ styles.css      ← 视觉设计（配色取自 logo）
├─ .pages.yml      ← 后台配置：定义每个编辑器能填哪些字段
├─ 开始本地预览.bat  ← 双击启动本地服务器
└─ assets/
   ├─ logo.svg
   └─ favicon.svg
```

**为什么这样分**：以前文案写死在 HTML 里，加一段话要改代码；现在文案是数据，
页面是模板。别人加内容不碰 HTML，也不再需要我这样的角色在中间转一道。

## 三种修改方式

### 1. 后台可视化改（推荐给编辑者）

[Pages CMS](https://app.pagescms.org) 是托管服务，不需要自己部署：

1. 打开 https://app.pagescms.org
2. 用 GitHub 账号登录
3. 授权后选择仓库 `ltjcherry/comebench`
4. 左侧看到 **「页面内容」**，分成三段：

| 区块 | 能改什么 |
|---|---|
| ① 基本信息 | 标题、会议标签、一句话简介、作者、单位、链接、BibTeX |
| ② 首屏数字条 | 那几个大数字和说明 |
| ③ 页面模块 | **可以新增、删除、上下拖动排序** |

「③ 页面模块」是重点：点 **Add** 会让你选模块类型，填完就多一块；拖动可以调整
顺序；删掉就整块消失。保存后自动提交到 GitHub，Pages 约 1 分钟自动重建。

> **给编辑者的权限**：对方需要是这个仓库的协作者。
> 仓库 → Settings → Collaborators → Add people，输入对方的 GitHub 用户名。

### 2. 直接改 content.json

适合只改几处文字。文件是纯 JSON，改完保存即可。注意：

- 用 UTF-8 保存，不要用记事本另存成 ANSI，中文会乱码
- 支持极简 Markdown：`**粗体**`、`*斜体*`、`` `代码` ``
- 改完用 `开始本地预览.bat` 检查，没问题再提交

### 3. 让 AI 改

把 content.json 或一句话需求发过来即可。

## 本地预览

双击 **`开始本地预览.bat`**，会自动起服务并打开 http://127.0.0.1:5500

必须用服务器打开，**不能直接双击 index.html**——浏览器会以 `file://` 协议
拦截 `content.json` 的读取，页面会显示一段提示而不是内容。

## 首屏背景视频

首屏可以放一段循环播放的视频（参考 OpenHLM 的做法）。在 `content.json` 里：

```json
"hero": {
  "video": "assets/openhlm-hero.mp4",
  "poster": "assets/hero-poster.jpg",
  "stats": [ … ]
}
```

- `video` 留空 → 自动退回渐变背景（不会报错）
- `poster` → 视频加载前显示的静态图，建议用 ffmpeg 抽第一帧：

  ```powershell
  ffmpeg -ss 1 -i assets/your-video.mp4 -frames:v 1 -q:v 3 assets/hero-poster.jpg
  ```

- 视频会被裁剪铺满（`object-fit: cover`），建议 **16:9、1280×720、H.264、无音轨**，
  文件控制在 10 MB 以内。
- 开启了 `autoplay muted loop playsinline`，浏览器才会允许静音自动播放。
- 系统开启「减少动态效果」时，视频会被隐藏，只显示封面图。

> 深色视频上用的是 `assets/logo-light.svg`（白色图形 + 保留渐变字标），
> 浅色背景用 `assets/logo.svg`，渲染器会自动切换。

## 模块类型

渲染器认识 6 种模块，都在 `render.js` 的 `RENDERERS` 里：

| type | 长什么样 | 主要字段 |
|---|---|---|
| `overview` | 左文字 + 右侧信息卡 | paragraphs, aside |
| `cards` | 三列特性卡片 | items(icon/title/text) |
| `tasks` | 任务卡片网格 + 「更多」卡 | items(tag/tagColor/title/text) |
| `videos` | 任务演示视频 + 层级标注 | items(tag/title/meta/prompt/video/poster/subtasks/listLabel) |
| `table` | 榜单表格 | columns, rows, note |
| `code` | 深色代码块（可带步骤列表） | filename, code, steps |

### `videos` 模块的播放行为

- 卡片**滚动到 55% 以上可见时自动播放**，滚出视口自动暂停（不会三个视频同时跑）
- 只加载封面图，`preload="none"`，滚到了才开始拉视频流，首屏不为它付流量
- 用户手动暂停后不会再被自动播放打断，直到他自己点播放
- 系统开启「减少动态效果」时，退化为纯点击播放

**想加一种新模块**（比如「图片墙」「时间线」「视频嵌入」）：在
`render.js` 的 `RENDERERS` 里加一个函数，再在 `.pages.yml` 的 `blocks`
里加一段字段定义。两处各加一段，后台立刻出现这个新类型。

## 发布

推送到 GitHub 后，仓库 → Settings → Pages：

- Source: `Deploy from a branch`
- Branch: `main`，文件夹 `/ (root)`
- Save

网址：**https://ltjcherry.github.io/comebench/**

> 注意：Pages 的 CDN 会缓存约 10 分钟，改动不会立刻在所有网络生效。

## 已知取舍

- **页面需要 JavaScript**。内容在加载后才渲染，所以搜索引擎和社交平台的抓取
  可能拿不到正文。`<title>` 和 og 标签是写死在 index.html 里的，需要改标题时
  手动改那几行。
- 换来的是**零构建**：没有 CI、没有构建失败、编辑器改错也只会显示错文字，
  不会让站点挂掉。对非技术编辑者更友好。

如果以后需要更好的 SEO，可以加一步静态生成（11ty/Astro + GitHub Actions），
把 content.json 预渲染成 HTML，结构不用变。
