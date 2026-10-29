# 🎵 MusicFree 插件订阅站（同款搭建包）

复刻自 [hebijunge.github.io/musicfree-plugins](https://hebijunge.github.io/musicfree-plugins/) 的整套机制：
**一个 `plugins.json` 数据源 → 页面自动渲染插件列表 → 多条订阅通道一键分发**。
当前仓库内为**示例数据**（占位域名 + 示例插件骨架），替换为你的真实插件后即可对外发布。

---

## 一、目录结构

```
musicfree-plugins-site/
├── index.html               # 主页面：自动读取 plugins.json 渲染，含搜索/复制/多通道
├── plugins.json             # ★ 唯一需要人工维护的数据源（MusicFree 标准订阅清单）
├── plugins-fastly.json      # 备用通道 · fastly 节点（jsDelivr CDN）    ┐
├── plugins-gcore.json       # 备用通道 · gcore 节点（jsDelivr CDN）     │ 由 generate-channels.ps1
├── plugins-ghproxy.json     # 国内加速 · ghproxy 代理                  │ 自动生成，无需手改
├── plugins-proxy-01.json    # 国内代理 · 示例（替换成实测代理即可）      ┘
├── plugins/                 # 插件 JS 存放目录
│   ├── sample-music-source.plugin.v1.0.0.js     # 示例插件（最小骨架）
│   └── sample-radio-source.plugin.v1.0.0.js     # 示例插件（最小骨架）
├── generate-channels.ps1    # 多通道清单自动生成器（命令行 / GUI 双模式）
├── 部署上线.ps1             # 方案A：安装 Git 后，一键探测分支→同步配置→推送上线
├── 上传上线-无git.ps1        # 方案B：不装 Git，用 GitHub 令牌直接上传上线
└── README.md                # 本文件
```

## 二、它是怎么工作的（机制原理）

1. **MusicFree 客户端**（开源播放器）打开「插件设置 → 订阅」，粘贴任一清单地址。
2. 客户端请求该地址，拿到 `{"desc":"0.1.0","plugins":[{name,url,version},…]}` 清单。
3. 按清单里的 `url` 逐个下载插件 JS 到本地并加载，App 内即可使用这些音源。
4. **页面**用 `fetch('plugins.json')` 读同一份清单渲染卡片，所以「新插件加进 plugins.json 就自动出现在页面」。
5. **多通道**：同一份插件列表，把每个插件的下载地址换成不同前缀（GitHub Pages / jsDelivr CDN / ghproxy / 国内代理），生成多份内容一致的清单，解决 GitHub 在国内访问慢/不稳定的问题。

## 三、快速开始（部署到 GitHub Pages，约 5 分钟）

### 第 1 步：确认配置（本项目已配置为 `liaowan001/musicfree-plugins`）

**① `index.html`** 顶部 `CONFIG` 对象（已填好，只需确认分支）：

```js
var CONFIG = {
  githubUser: 'liaowan001',
  githubRepo: 'musicfree-plugins',
  branch: 'master',           // 如果你的仓库主分支是 main，改成 'main'
  isDemo: true,               // 换成真实插件后改为 false
  ...
}
```

**② `generate-channels.ps1`** 顶部配置区（已填好，只需确认分支）：

```powershell
$Script:Config = @{
    GithubUser = 'liaowan001'
    GithubRepo = 'musicfree-plugins'
    Branch     = 'master'     # 与仓库主分支一致
    ...
}
```

### 第 2 步：生成多通道文件

```powershell
# 命令行模式
powershell -ExecutionPolicy Bypass -File .\generate-channels.ps1

# 或图形界面模式（推荐新手）
powershell -ExecutionPolicy Bypass -File .\generate-channels.ps1 -Gui
```

### 第 3 步：把项目推送到仓库（任选一种方案）

**方案 A（推荐，装 Git，一劳永逸）**
1. 安装 Git for Windows：https://git-scm.com/download/win（默认选项一路下一步）
2. 在项目目录运行一键部署脚本（自动探测分支、同步配置、生成通道、提交推送）：
   ```powershell
   powershell -ExecutionPolicy Bypass -File .\部署上线.ps1
   ```
3. 推送时如弹出 GitHub 登录窗口，按提示完成登录即可。脚本最后会输出订阅地址和 Pages 开启指引。

**方案 B（不想装 Git，用令牌直传）**
1. 生成一个仓库访问令牌（二选一）：
   - 经典令牌：https://github.com/settings/tokens → Generate new token (classic) → 勾选 `repo`
   - 细粒度令牌（推荐）：https://github.com/settings/tokens?type=beta → Repository access 选本仓库 → Permissions → Contents → Read and write
2. 运行上传脚本，按提示粘贴令牌（不会显示，回车确认）：
   ```powershell
   powershell -ExecutionPolicy Bypass -File .\上传上线-无git.ps1
   ```
3. 全部文件自动上传到 main 分支，等 1 分钟 Pages 自动构建即可访问。

> 手动方式备选：git init → git add . → git commit → git remote add origin
> https://github.com/liaowan001/musicfree-plugins.git → git branch -M main → git push -u origin main

### 第 4 步：开启并访问 GitHub Pages

仓库页 → **Settings → Pages**（你已配置为 main / (root)）→ 上传内容后自动生效。约 1 分钟后访问：

```
https://liaowan001.github.io/musicfree-plugins/
```

订阅地址即：`https://liaowan001.github.io/musicfree-plugins/plugins.json`

## 四、日常维护：如何加一个新插件

1. 把插件 JS 放进 `plugins/` 目录（命名建议：`英文名-source.plugin.v版本号.js`）。
2. 在 `plugins.json` 的 `plugins` 数组末尾追加一条：

   ```json
   {
     "name": "插件显示名",
     "url": "https://你的用户名.github.io/仓库名/plugins/你的插件.js",
     "version": "1.0.0"
   }
   ```

3. 运行 `generate-channels.ps1` 同步所有通道文件。
4. `git add . && git commit -m "add 插件名" && git push` → 页面自动更新，订阅用户下次刷新即收到。

> 想要更多国内代理通道？在 `generate-channels.ps1` 的 `Channels` 数组里复制一行，把
> `File` 改成 `plugins-proxy-02.json`、把 `PROXY_HOST` 换成你实测可用的代理域名即可。

## 五、示例数据 → 真实数据替换清单

用户名/仓库名已配置为 `liaowan001/musicfree-plugins`，上线前还需要：

| # | 位置 | 替换内容 |
|---|------|----------|
| 1 | `index.html` CONFIG | 确认 `branch` 与仓库主分支一致；替换插件后把 `isDemo` 改为 `false` |
| 2 | `generate-channels.ps1` | 确认 `Branch` 与仓库主分支一致 |
| 3 | `plugins.json` | 删掉全部「示例·」条目，换成你的真实插件 |
| 4 | `plugins/` 目录 | 删掉 `sample-*` 示例文件，换成你的插件 JS |
| 5 | 重新运行生成脚本 | 同步各通道文件 |

## 六、常见问题

**Q：本地双击 index.html 打开，插件列表能显示但链接打不开？**
正常。本地 `file://` 下 `fetch('plugins.json')` 会被浏览器拦截，页面自动改用内置兜底数据渲染（功能演示用）；插件下载链接指向线上域名，部署后才会真实可用。完整预览建议部署后访问线上地址。

**Q：MusicFree 订阅提示 Network Error？**
换一条通道地址重试（页面上提供多条）。国内网络环境下优先 `ghproxy` 或国内代理通道。

**Q：我的仓库主分支是 main 怎么办？**
`index.html` 的 CONFIG.branch、`generate-channels.ps1` 的 Branch、git 推送分支、GitHub Pages 选择的分支，四处保持一致为 `main` 即可。

**Q：代理通道怎么测速、怎么收录？**
自行用 `curl -o /dev/null -w "%{speed_download}" https://代理域名/https://raw.githubusercontent.com/…` 实测，把测速结果填进 `index.html` 通道配置的 `note` 字段（会显示在订阅行尾）。

## 七、合规提醒

- 插件 JS 请确保为**自有代码或已获授权**的内容；请勿发布未获授权的抓取、逆向或破解类插件。
- 示例插件 `sample-*.js` 仅演示 MusicFree 插件加载机制（空实现骨架），不含任何数据源逻辑。
- MusicFree 插件开发请参阅官方文档：https://musicfree.codelife.cc/
