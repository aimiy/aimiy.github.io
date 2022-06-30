---
layout: post
title:  typescript的配置文件tsconfig.json
date:   2021-01-21 11:00:00 +0800
categories: ts
tag: 笔记
---
* content
{:toc}

## TSConfig 前言

目录中的 TSConfig 文件表明该目录是 TypeScript 或 JavaScript 项目的根目录。 TSConfig 文件可以是 tsconfig.json 或 jsconfig.json，它们的配置项和行为相同。

TS使用tsconfig.json作为配置文件，主要包含两块内容

>指定待编译的文件

>定义编译选项

## 一个简单示例

`compilerOptions`用来配置编译选项，`files`用来指定待编译文件。
这里的待编译文件是指入口文件，任何被入口文件依赖的文件

```json
{
  "compilerOptions": {
    "module": "esnext",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
  },
  "files": ["app.ts"]
}
```

也可以使用`include`和`exclude`来指定和排除待编译文件：

```json
{
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "exclude": ["node_modules"],
}
```

## 配置简介

参考官方文档[tsconfig](https://www.typescriptlang.org/zh/tsconfig)

名称|含义|属性|属性含义
---|---|---|---
module|生成的JavaScript模块形式|none|
|||commonjs|
|||amd|
|||system|
|||umd|
|||es6|
|||es2015|
|||esnext|
nolmplicitAny|存在隐式any时抛错|false（默认）
sourceMap|生成map文件|false（默认）
declaration|生成对应的.d.ts文件|false（默认）
