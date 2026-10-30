# 时空漫游（拆分版）

## 文件结构

```text
时空漫游/
├── index.html          # 页面结构
├── css/
│   └── style.css       # 页面样式
├── js/
│   ├── data.js         # 历史事件、坐标、课程数据
│   └── app.js          # 页面交互与业务逻辑
└── README.md
```

## 如何运行

1. 将整个文件夹放在同一个目录中。
2. 使用浏览器打开 `index.html`。
3. 建议通过本地静态服务器访问，而不是直接双击文件。
4. Three.js 仍通过 CDN 加载，因此首次运行需要网络连接。
5. 原有 `media/` 目录（如果有课件图片、PPT 等）需要放在 `index.html` 同级目录，保持原路径不变。

## 本次拆分原则

- 保留原有页面结构、业务逻辑、数据内容和 localStorage 键名。
- 将 `<style>` 中的 CSS 提取到 `css/style.css`。
- 将历史事件、坐标和课程数据提取到 `js/data.js`。
- 将时间轴、太阳系动画、3D 场景、课程中心、反馈、导览和媒体展示逻辑放入 `js/app.js`。
- 将首页两个内联 `onclick` 改为通过 JavaScript 绑定事件。

## 后续建议

本次是第一阶段的结构拆分，不是完整的架构重写。下一阶段可以继续把 `app.js` 拆成：
- `timeline.js`：时间轴与播放控制
- `solar-system.js`：Canvas 太阳系动画
- `globe.js`：Three.js 天体渲染
- `course.js`：课程、测试与学习评估
- `feedback.js`：课程反馈与需求反馈
- `tour.js`：新手导览
- `media.js`：图片、视频、课件展示

进一步拆分时，建议逐步引入 ES Modules（`type="module"`），明确模块之间的 `import/export`，避免继续依赖全局变量。
