# 🎵 SvelteKit 黑胶音乐播放器

一个使用 SvelteKit 构建的交互式、视觉丰富的音乐播放器。浏览标志性专辑的合集，查看其详细信息，并播放示例曲目——所有内容都以动画黑胶唱片的形式呈现。

## 🚀 功能特色

- **动画启动画面**：以打字机效果欢迎用户。
- **黑胶唱片播放器**：专辑以旋转的黑胶形式可视化，叠加显示流派、年代和流行度。
- **专辑轮播**：浏览并选择专辑合集。
- **详细专辑信息**：查看艺术家、年份、流派、排名、描述和曲目列表。
- **音频播放**：每张专辑均支持播放、暂停和停止控制。
- **响应式设计**：在桌面和移动端均可无缝使用。
- **数据驱动**：专辑数据从 CSV 文件加载，便于更新。

## 🗂️ 项目结构

```
Proyecto2/
├── public/              # 静态资源（图片、图标）
├── src/
│   ├── routes/
│   │   ├── +layout.svelte   # 应用布局
│   │   └── +page.svelte     # 主要音乐播放器界面与逻辑
│   └── lib/                 # （可选）共享组件/工具
├── static/
│   ├── covers/          # 专辑封面图片
│   ├── data/albums.csv  # 专辑元数据（id, title, artist, ...）
│   ├── music/           # 每张专辑的音频文件
│   ├── overlays/        # UI 叠加层（年代、唱纹、排名）
│   └── vinyls/          # 按流派分类的黑胶底图
├── package.json         # 项目元数据与脚本
├── svelte.config.js     # SvelteKit 配置
└── vite.config.ts       # Vite 配置
```

## 📦 安装与开发

1. **安装依赖：**
   ```bash
   npm install
   # 或
   pnpm install
   # 或
   yarn install
   ```

2. **运行开发服务器：**
   ```bash
   npm run dev
   # 或
   npm run dev -- --open  # 自动在浏览器打开
   ```

3. **生产环境构建：**
   ```bash
   npm run build
   ```

4. **预览生产构建：**
   ```bash
   npm run preview
   ```

## 📝 数据与自定义

- **添加/更新专辑：**
  - 编辑 `static/data/albums.csv` 以添加新专辑或更新现有专辑。
  - 将对应的封面图片放入 `static/covers/`，音频文件放入 `static/music/`。
- **流派、年代与叠加层：**
  - 在 `static/vinyls/` 中添加或修改黑胶底图。
  - 年代和唱纹的叠加层位于 `static/overlays/`。

## 🛠️ 脚本命令

- `npm run dev` — 启动开发服务器
- `npm run build` — 生产环境构建
- `npm run preview` — 预览生产构建
- `npm run lint` — 代码检查
- `npm run format` — 代码格式化

## 📚 技术栈

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)

## 📄 许可证

MIT（或在此处注明您的许可证）

---

> 灵感来源于黑胶之美与 Svelte 的强大。