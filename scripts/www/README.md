# 困难选择器 - UniApp 项目

一款有趣的英雄随机选择工具，支持滚珠转盘动画效果。

## 功能特性

- 🎲 **随机选择** - 滚珠转盘式随机选择
- 📦 **模块管理** - 支持多模板导入导出
- 🎨 **主题切换** - 多种主题风格
- ✨ **动画效果** - 流畅的滚珠动画
- 📱 **多端适配** - 支持 H5、小程序、App

## 项目结构

```
uniapp-project/
├── pages/
│   ├── home/          # 首页
│   ├── select/        # 选择页（滚珠转盘）
│   ├── modules/       # 模块管理
│   └── settings/      # 设置页面
├── static/            # 静态资源
│   ├── styles/        # 全局样式
│   └── tabbar/        # TabBar 图标
├── App.vue            # 应用入口
├── main.js            # 主入口
├── manifest.json      # 应用配置
├── pages.json         # 路由配置
└── package.json       # 依赖配置
```

## 快速开始

### 安装依赖

```bash
npm install
```

### 开发预览

```bash
# H5 预览
npm run dev:h5

# 微信小程序
npm run dev:mp-weixin

# App 开发
npm run dev:app
```

### 构建发布

```bash
# 构建 H5
npm run build:h5

# 构建微信小程序
npm run build:mp-weixin

# 构建 App
npm run build:app
```

## 后端 API

项目需要后端服务支持，请确保后端服务运行在 `http://localhost:9091`

### API 接口

- `GET /api/v1/templates` - 获取模板列表
- `GET /api/v1/templates/:id` - 获取模板详情

## 打包 App

### 使用 HBuilderX

1. 下载 [HBuilderX](https://www.dcloud.io/hbuilderx.html)
2. 导入项目
3. 运行 → 运行到手机或模拟器
4. 发布 → 原生App云打包

### 使用 PakePlus

```bash
# 安装 PakePlus
npm install -g pake-plus-cli

# 打包 H5 为 App
pake-plus ./dist --platform android --name "困难选择器"
```

## 技术栈

- Vue 3
- UniApp
- SCSS
- Vite

## License

MIT
