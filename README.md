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

## 声明

页面内容为公开信息整理，Star 数为抓取时近似值，可能已经变化。
不构成对任何项目的推荐或担保，各项目功能与限制以仓库最新说明为准。
