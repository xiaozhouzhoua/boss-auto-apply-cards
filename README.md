# Boss 直聘自动投递 — GitHub 项目调研

一份单文件 HTML 调研页，整理 GitHub 上可用于「BOSS 直聘自动投递简历」的开源项目。

**在线查看 → https://xiaozhouzhoua.github.io/boss-auto-apply-cards/**

## 文件

| 文件 | 说明 |
|---|---|
| `boss-auto-apply-cards.html` | 全部内容。双击即可用浏览器打开，无构建、无依赖、无外部请求 |
| `.github/workflows/pages.yml` | 推送到 `main` 时自动发布站点，将上面的文件复制为站点入口 `index.html` |

> 想在浏览器里看到渲染效果，用上面的在线地址。直接打开仓库里的 `.html`
> 只会看到源码，因为 GitHub 对这类文件返回 `text/plain`。

## 内容

收录 9 个项目，分为本地程序、浏览器扩展、命令行三类：

- **速览对比表** — 形态 / 技术栈 / 核心机制 / 许可证 / Star
- **9 个项目详情** — 核心特点、配置要点、限制与风险、上手命令
- **推荐上手顺序** — 5 步，从最省事的方案到放开限制正式跑
- **通用风险与注意事项** — 7 条，含账号封禁风险与许可证说明

交互：分类筛选、关键词搜索（匹配项目名 / 语言 / 特点全文）、滚动进度条、
卡片入场动画。

## 设计

暗色 Apple 毛玻璃风格。`backdrop-filter` 必须有背景内容才有效果，因此页面前置了
一个会被模糊的动态光场：5 层固定径向渐变 + 3 个缓慢漂移的光斑 + 一层噪点，
玻璃面板滚动时透出的颜色会持续变化。

玻璃材质由三层叠加而成：

1. 渐变描边（`mask-composite: exclude` + `padding:1px`），模拟左上光源
2. 顶部内高光（`inset 0 1px 0`），制造厚度感
3. 半透明底色 + `saturate(190%)`，让透上来的背景色更通透

## 无障碍与降级

| 场景 | 处理 |
|---|---|
| 不支持 `backdrop-filter` | `@supports not` 回退为不透明实色面板 |
| 系统开启「降低透明度」 | `prefers-reduced-transparency` 关闭模糊 |
| 系统开启「减弱动态效果」 | 停止光斑动画、取消入场位移、压缩过渡时长 |
| 窄屏（≤900px / ≤640px） | 卡片转为单列，侧栏元数据折叠为一行 |

## 预览

**在线**：https://xiaozhouzhoua.github.io/boss-auto-apply-cards/

**本地**：直接双击 `boss-auto-apply-cards.html`，或用任意静态服务器：

```bash
python -m http.server 8000
# 然后访问 http://localhost:8000/boss-auto-apply-cards.html
```

修改后推送到 `main`，Pages 会自动重新发布：

```bash
git add -A && git commit -m "更新调研内容" && git push
```

也可在仓库 Actions 页面手动触发 `Publish page` 工作流。

## 这个站点是怎么跑起来的

先说清楚网址本身是怎么来的：

```
https://xiaozhouzhoua.github.io/boss-auto-apply-cards/
        └──── 子域名，GitHub 按账号名免费分配 ────┘  └── 仓库名 ──┘
```

`github.io` 是 GitHub 自己的域名，不是谁申请的。它给 `*.github.io` 配了泛解析，
所有用户的子域名都指向同一批服务器；请求到达时，服务器靠 **`Host` 请求头**
判断该返回哪个仓库的内容。

顺带一个安全设计：`github.io` 在 [Public Suffix List](https://publicsuffix.org/) 上，
所以浏览器把每个 `*.github.io` 当作**互相隔离的独立站点**。否则任何人都能用
自己的子域名给 `github.io` 设一个 cookie，去影响别人的页面。

`.github/workflows/pages.yml` 不是程序，是一份**交给 GitHub 的待办清单**。它只回答三个问题：

```
什么时候干？  →  push 到 main 时
用什么干？    →  借一台 GitHub 的临时 Linux 机器
具体干什么？  →  复制文件 → 打包 → 发布
```

实际链路：

```
git push
  └─ GitHub 读到 pages.yml，匹配到 on.push
      └─ 开一台全新 Ubuntu 机器（空的）
          ├─ actions/checkout            把仓库代码下载进去
          ├─ run: cp … _site/index.html  组装发布目录
          ├─ actions/upload-pages-artifact  打包上传
          └─ actions/configure-pages / deploy-pages  验签并发布
              └─ 绑定到 xiaozhouzhoua.github.io/boss-auto-apply-cards/
```

全程约 20 秒，之后机器销毁。内容由 GitHub 的 CDN 提供，**本地电脑不参与**。

### 为什么一个文件就够了

因为 `uses:` 那几行是**复用官方封装好的现成 action**。「部署到 Pages」这件事背后涉及
OIDC 令牌申请、签名验证、制品存储、部署记录提交，自己实现要几百行且容易出错；
官方把它封成 `actions/deploy-pages@v4`，这里一行调用即可。

复杂度没有消失，是被封装起来、由大量用户在共用了。

### 关于 `index.html`

Web 服务器的通用约定：请求以 `/` 结尾的目录时，会查找默认文档 `index.html`。
所以站点入口**必须**叫这个名字。

本仓库有意保留 `boss-auto-apply-cards.html` 这个有信息量的原名（直接在仓库里看名字就知道是什么），
由工作流在发布时复制为 `index.html` 来满足约定。

工作流里的 `.nojekyll` 是零成本保险：告诉 Pages 不要用 Jekyll 处理，原样发布。
附带的用处是防止下划线开头的路径被 Jekyll 忽略。

### 两项权限的作用

```yaml
permissions:
  pages: write      # 允许发布 Pages
  id-token: write   # 允许申请 OIDC 令牌
```

`id-token` 不是多余的。制品内容来自仓库、而仓库可能被恶意 PR 污染，所以 GitHub
不会因为「有人上传了东西」就无条件发布——`deploy-pages` 必须拿 OIDC 令牌证明
「我确实是这个仓库、这个工作流、这次运行」，验签通过才允许上线。

### 其实可以不用工作流

如果不在意文件名，有更简单的做法：把文件改名为 `index.html`，
然后在 Settings → Pages 里把 Source 设为 `main` 分支根目录即可，**不需要任何工作流**。

用工作流是为了保住原来的文件名。判断标准很简单：

- 不在意文件名 → Settings 里点两下就够
- 想保留有意义的文件名 → 用工作流多做一层转换

## 搭建过程中踩到的坑

记录于 2026-10-05 的实际配置过程。本机处于代理环境（系统代理指向本地端口），
下面几条对同类环境有参考价值。

### `SEC_E_NO_CREDENTIALS` 是沙箱伪故障，不是 Windows 坏了

排查时 `curl.exe`、PowerShell/.NET、`git clone` 全部报：

```
schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS
```

一度判断为「Windows Schannel 子系统故障」，**这个结论是错的**。
放松执行沙箱后，同样的 `curl.exe https://api.github.com/zen` 立刻返回 `http=200`。

真实原因是受限环境不允许进程访问系统凭证库，**与网络和 Windows 本身都无关**。
当时写入的变通配置因此是多余的，可以清掉：

```bash
git config --global --unset http.sslBackend
```

教训：遇到 `AcquireCredentialsHandle` 失败，先确认是不是运行环境受限，
不要急着换 TLS 后端。

### 直连 GitHub 不通，必须走代理

用**真实仓库路径**对照测试（这一点很关键）：

| 方式 | 结果 |
|---|---|
| 直连 `github.com` | 超时，21 秒后 `Could not connect to server` |
| 走本地代理 | 成功 |

判定方法是做对照实验：显式清空代理后立刻复现超时，加回后即通。

代理配置（`http.proxy` / `https.proxy` 走全局，其余工具走用户级环境变量）：

```bash
git config --global http.proxy  http://127.0.0.1:10808
git config --global https.proxy http://127.0.0.1:10808
git config --global http.noProxy "localhost,127.0.0.1,::1,*.local,192.168.*,10.*,172.16.*"
```

`noProxy` 保证本地和内网地址直连，不会被绕进代理。

### 测试方法本身也会骗人

第一轮连通性测试报了 4 个 FAIL，看起来像代理配置无效。实际是我测错了：

```bash
git ls-remote https://github.com          # 域名根上没有仓库，404 是必然的
git ls-remote https://github.com/user/repo.git   # 这样才对
```

用正确的路径重测后全部通过。**结论反转往往来自测量方法，而不是被测对象。**

### 老终端不继承新写入的环境变量

配置后用 `curl` 测 `raw.githubusercontent.com` 报 `Could not resolve host`，一度以为域名被墙。
实际是当前终端进程仍是旧会话，没有继承刚写入的用户级变量；注入后立刻 `http=200`。

跨会话生效需要**重开终端**。

### `index.html` 这个约定会咬人

- 直接发布仓库根目录时，因为入口文件不叫 `index.html`，访问 `/boss-auto-apply-cards/` 会 404
- 打开仓库里的 `.html` 看到的是**源码而不是页面**，因为 GitHub 返回 `Content-Type: text/plain`
- 改动 HTML 后站点不会立即变化，需要等 Pages 重新发布

### 凭据是明文存储的

`gh auth login` 会把 git 凭据助手设为 `credential.helper store`，
token 以明文存放在用户目录。能用，但在意的话应换成系统凭据管理器：

```bash
git config --global credential.helper manager
```

### 权限声明不是走过场

最初忘记先启用 Pages，工作流会在 `deploy` 阶段失败。
发布型工作流还需要显式声明 `pages: write` 和 `id-token: write`，
否则同样是 403 —— 见上一节的说明。

## 声明

页面内容为公开信息整理，Star 数为抓取时近似值，可能已经变化。
不构成对任何项目的推荐或担保，各项目功能与限制以仓库最新说明为准。
