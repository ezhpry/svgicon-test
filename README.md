# svgicon-test

本项目用于学习 Vite + Vue 3 项目中 SVG Icon 组件的封装方案。

## 架构概述

基于 **vite-plugin-svg-icons** 实现 SVG 图标管理，整体分为三层：

1. **构建层（Vite 插件）** — 扫描 `src/assets/icons/` 目录下的 `.svg` 文件，自动生成 SVG Sprite（`<symbol>` 合集）并注入到页面 `<body>` 中
2. **注册层（虚拟模块）** — `main.ts` 中引入 `virtual:svg-icons-register`，将 Sprite 注册到 DOM
3. **组件层（SvgIcon.vue）** — 封装 `<svg><use></use></svg>` 的 Vue 组件，通过 `name` prop 引用对应的 `<symbol>`

## 背景知识

### SVG Sprite 是什么？

一个典型的 `.svg` 文件长这样：

```xml
<svg viewBox="0 0 24 24">
  <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z" />
</svg>
```

**SVG Sprite** 就是把多个 SVG 图标合并到一个大文件中。就像游戏里的精灵图（Sprite Sheet），把很多小图拼成一张大图，渲染时只取需要的那一小块。

在这个项目中，`vite-plugin-svg-icons` 插件会把 `icons/` 目录下所有 `.svg` 收集起来，拼成一个 Sprite 注入到页面的 `<body>` 中。

### `<symbol>` 是什么？

`<symbol>` 是 SVG 的一个标签，用来**定义可复用的图形模板**。它把每个图标的矢量路径包裹起来，但**不会自己渲染到页面上**——它只是模板，等着被引用。

```html
<!-- 插件生成的 Sprite（注入到 <body>） -->
<svg id="__svg__icons__dom__">
  <symbol id="icon-arrow" viewBox="0 0 24 24">
    <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z" />
  </symbol>
  <symbol id="icon-home" viewBox="0 0 24 24">
    <path d="..." />
  </symbol>
  <symbol id="icon-settings" viewBox="0 0 24 24">
    <path d="..." />
  </symbol>
</svg>
```

可以这样理解：`<symbol>` 是"模板库"，光有模板不会显示任何东西。

### `<use>` 是什么？

`<use>` 是 SVG 的一个标签，用来**引用并实例化一个 `<symbol>`**。它通过 `href` 属性指向某个 symbol 的 `id`，把模板"实例化"并渲染出来。

```html
<!-- 引用 #icon-arrow 这个 symbol，渲染出箭头图标 -->
<svg>
  <use href="#icon-arrow" />
</svg>
```

可以这样理解：`<symbol>` 是定义（define），`<use>` 是使用（use）。

### 三者关系一句话总结

```
多个 .svg 文件  ──打包成──▶  SVG Sprite（含多个 <symbol> 模板）
                                      │
                          <use href="#某个symbol">  ◀── 按需引用其中一个
```

> **类比：** Sprite 是一本贴纸书，每个 `<symbol>` 是书里的一张贴纸，`<use>` 是把贴纸揭下来贴到页面上的动作。

## 流程简述

从放入 SVG 文件到页面渲染，各环节的对应关系如下：

```
src/assets/icons/arrow.svg          ← 把 .svg 文件放在这里
        │
        ▼
vite.config.ts 中 createSvgIconsPlugin 扫描该目录
  symbolId: 'icon-[dir]-[name]'      ← id 格式由这里定义
        │
        ▼
生成 <symbol id="icon-arrow">        ← 插件自动生成并注入到 <body>
        │
        ▼
<SvgIcon name="arrow" prefix="icon" />   ← 组件拼接出 symbolId
        │
        ▼
<use href="#icon-arrow" />          ← 最终引用对应的 symbol
```

**关键映射：**

| 文件路径 | `symbolId` 配置 | Props | 生成的 id | `<use>` 引用 |
|----------|----------------|-------|-----------|-------------|
| `icons/arrow.svg` | `icon-[dir]-[name]` | `prefix="icon"` `name="arrow"` | `icon-arrow` | `#icon-arrow` |
| `icons/home.svg` | `icon-[dir]-[name]` | `prefix="icon"` `name="home"` | `icon-home` | `#icon-home` |
| `icons/foo/bar.svg` | `icon-[dir]-[name]` | `prefix="icon"` `name="foo/bar"` | `icon-foo-bar` | `#icon-foo-bar` |

> **记住一条规则：** `name` prop 填的是 SVG 文件名（去掉 `.svg`），`prefix` 与 `vite.config.ts` 中 `symbolId` 的前缀保持一致，两者拼起来就是 `<symbol>` 的 id。

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
