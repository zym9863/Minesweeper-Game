# 扫雷游戏

[English](README.md) | [中文](README_zh.md)

一个使用 Vue 3、TypeScript 和 Vite 构建的现代扫雷游戏实现。

## 功能特点

- 经典扫雷游戏玩法
- 多种难度级别（初级、中级、高级）
- 自定义游戏配置
- 计时器和地雷计数器
- 首次点击保护（第一次点击永远不会是地雷）
- 现代化界面和动画效果
- 响应式设计

## 技术栈

- **Vue 3** 使用组合式 API 和 `<script setup>` 语法
- **TypeScript** 提供类型安全
- **Vite** 用于快速开发和优化构建
- 自定义 CSS 变量实现主题化

## 项目结构

```
src/
├── assets/         # 图标和图片
├── components/     # Vue 组件
│   ├── Cell.vue    # 单元格组件
│   ├── GameBoard.vue # 游戏主面板
│   └── GameControls.vue # 游戏控制器（计时器、重启、难度选择）
├── App.vue         # 主应用组件
├── main.ts         # 应用入口点
└── style.css       # 全局样式和 CSS 变量
```

## 游戏玩法

- 左键点击揭示单元格
- 右键点击放置/移除旗帜
- 数字显示相邻单元格中的地雷数量
- 标记出所有地雷即可获胜！

## 开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

## 许可证

MIT
