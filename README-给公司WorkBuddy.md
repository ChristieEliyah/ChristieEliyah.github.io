# 作品集部署指令

## 你是谁
你是公司的 WorkBuddy，用户刚才在家里改好了作品集文档，现在到你这里了。这个文件夹就是打包好的部署包，你照着下面的指令做就行。

## 仓库信息
- GitHub 仓库：`ChristieEliyah/ChristieEliyah.github.io`
- GitHub Pages 地址：https://christieeliyah.github.io/
- 仓库里已有的文件：`index.html`（首页，不用改）

## 要做什么

把本文件夹里的 4 个文件推到 GitHub 仓库根目录，覆盖同名文件：

| 文件 | 说明 | 仓库里是否已存在 |
|------|------|----------------|
| `fps-design-doc.html` | 设计文档（网页版） | 已存在，覆盖 |
| `fps-design-doc.md` | 设计文档（源文件） | 已存在，覆盖 |
| `fps-update-log.html` | 开发日志（网页版） | 新增 |
| `fps-update-log.md` | 开发日志（源文件） | 新增 |

## 具体步骤

1. 找到这个文件夹的路径（用户会告诉你，或者你搜 `作品集部署包`）
2. git clone 仓库到本地（或用户已有 clone）
3. 把这 4 个文件复制到仓库根目录
4. 在 `index.html` 里加一个链接指向 `fps-update-log.html`（如果还没有的话）
5. git add + commit + push

## index.html 怎么改

打开 `index.html`，找到指向 `fps-design-doc.html` 的链接，在旁边加一个：

```html
<a href="fps-update-log.html">开发日志</a>
```

具体样式参照已有的 `fps-design-doc.html` 链接的写法，保持一致。

## commit message

```
Update design doc to v0.42 + add update log
```

## 文档内容概要（不用改，只是让你知道里写了啥）

### fps-design-doc（设计文档 v0.42）
- 项目名称：仿COD UE FPS 射击游戏 Demo
- 版本规划：v0.1(3C) → v0.2(射击) → v0.3(外围) → v0.4(靶场) → v0.41(补弹) → v0.42(增加小球) → v1.00(Aimlab) → v2.00(滑铲蹲下白盒)
- 8 个系统：玩家移动、输入系统、武器系统、表面命中反馈、弹药管理、弹药补给、靶场小球系统、计分计时
- 核心玩法：当前仅射击小球，v1.00 目标实现大部分 Aimlab 功能

### fps-update-log（开发日志）
- v0.1 — 3C 开发
- v0.2 — 射击系统开发
- v0.3 — 外围系统开发
- v0.4 — 靶场初见雏形
- v0.41 — 补弹功能
- v0.42 — 增加小球（当前版本）

## 注意事项
- 不要改 index.html 里已有的链接，只加新的
- 不要改其他文件
- 推完之后访问 https://christieeliyah.github.io/ 确认首页能跳到两个文档
