# 翻页时钟 · Flip Clock

> 一款极简高颜值的网页翻页时钟，手机横放即可当桌面摆件。

## 项目定位

Minimalism（极简主义）与拟物翻页（Skeuomorphic Flip）的结合。大面积留白、高对比度字体、细腻的阴影与丝滑的3D翻页动效。

## 核心功能

- **3D 翻页动效** — 模拟物理翻页时钟的折叠下落效果，带重力回弹
- **时间格式** — 支持 24 小时制 / 12 小时制（含 AM/PM）
- **秒针开关** — 可选显示秒，翻页动态感更强
- **三套主题** — 暗黑纯粹（默认）、极简包豪斯、复古摩登
- **响应式布局** — 手机 / iPad / 宽屏显示器自动等比例缩放，始终居中
- **全屏模式** — 双击或一键全屏，隐藏浏览器边框
- **翻页音效** — 可选开启轻微"咔哒"机械声
- **日期星期** — 可显示/隐藏当前日期与星期

## 技术栈

- HTML5 / CSS3 / Vanilla JavaScript（纯原生，无构建步骤）
- CSS 3D Transform（`transform-style: preserve-3d`）实现翻页纵深感
- HTML5 Audio API 驱动翻页音效
- 移动优先，PWA manifest 支持

## 项目结构

```
.
├── index.html              # 主页面（时钟 + 全部逻辑）
├── favicon.svg             # 站点图标
├── manifest.webmanifest    # PWA manifest
├── Flip_Clock_Requirements.md  # 需求文档
├── README.md               # 本文档
└── .agent/
    └── skills/
        └── 发布/
            └── SKILL.md    # 发布流程规范
```

## 使用方式

直接用浏览器打开 `index.html`，或部署到任意静态托管服务。

手机使用建议：
- 竖屏时自动旋转 90° 横幅向显示
- 双击屏幕切换全屏
- 边缘淡出控制面板（移动/点击空白处显示）

## 版本规范

`vX.Y.Z` — X: 0-9, Y: 0-99, Z: 1-99

## License

MIT