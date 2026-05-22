# svgicon-test

本项目用于学习 Vite + Vue 3 项目中 SVG Icon 组件的封装方案。

## 架构概述

基于 **vite-plugin-svg-icons** 实现 SVG 图标管理，整体分为三层：

1. **构建层（Vite 插件）** — 扫描 `src/assets/icons/` 目录下的 `.svg` 文件，自动生成 SVG Sprite（`<symbol>` 合集）并注入到页面 `<body>` 中
2. **注册层（虚拟模块）** — `main.ts` 中引入 `virtual:svg-icons-register`，将 Sprite 注册到 DOM
3. **组件层（SvgIcon.vue）** — 封装 `<svg><use></use></svg>` 的 Vue 组件，通过 `name` prop 引用对应的 `<symbol>`

## 文件结构

```
src/
├── assets/icons/       ← 存放 .svg 图标文件
├── components/
│   └── SvgIcon.vue     ← 封装的 SVG Icon 组件
├── main.ts             ← 入口，引入 virtual:svg-icons-register
├── App.vue             ← 使用示例
vite.config.ts          ← Vite 插件配置
env.d.ts                ← TypeScript 类型声明（含 *.vue 模块声明）
```

## 组件用法

```vue
<script setup lang="ts">
import SvgIcon from './components/SvgIcon.vue'
</script>

<template>
  <!-- 基础用法：name 对应 icons 目录下的文件名 -->
  <SvgIcon name="arrow" />

  <!-- 自定义尺寸（Tailwind） -->
  <SvgIcon name="arrow" customCss="size-8" />
  <SvgIcon name="arrow" customCss="w-6 h-6" />

  <!-- 自定义前缀 -->
  <SvgIcon name="arrow" prefix="icon" />
</template>
```

### Props

| Prop | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | `string`（必填） | — | 图标名称，对应 SVG 文件名 |
| `prefix` | `string` | `'icon'` | symbol 前缀，对应 `symbolId` 中的前缀 |
| `customCss` | `string` | `''` | 自定义 CSS 类名，用于控制尺寸/颜色等 |

### symbolId 规则

`symbolId` 由 Vite 插件中的 `symbolId` 配置 + 文件名决定。默认格式为：

```
#icon-文件名
```

例如 `src/assets/icons/arrow.svg` → `<use href="#icon-arrow" />`。

## 依赖管理

```sh
pnpm i vite-plugin-svg-icons -D
```

## Project Setup

```sh
pnpm install
```

### Compile and Hot-Reload for Development

```sh
pnpm dev
```

### Type-Check, Compile and Minify for Production

```sh
pnpm build
```
