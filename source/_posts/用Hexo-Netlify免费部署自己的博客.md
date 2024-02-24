---
title: 用Hexo+Netlify免费部署自己的博客
date: 2024-02-24 13:08:49
categories: 
- 技术
tags:
- 博客
- Hexo
---

最近看到周围的朋友都搞了自己的博客，非常羡慕。然后上网搜了一下，发现了这么一个好东西：

**Hexo**：[hexo.io](https://hexo.io "点我前往Hexo官网")
<br><br>
这是什么？<br>
Hexo是一款基于Node.js的博客框架，上手快，简洁高效，使用Markdown来解析文章
<br><br><br>



## 准备工作
你需要安装以下软件包：<br>
``nodejs``<br>
``git``<br>

## 安装
首先安装hexo cli

```
npm i -g hexo-cli
```
下载太慢？先换个源吧
```
#该命令会把npm的下载源切换成国内的淘宝源
npm config set registry https://registry.npmmirror.com

##该命令会把npm的下载源切换回官方的源
npm config set registry https://registry.npmjs.org
```

## 开始使用
使用这个命令创建你的博客目录
```
hexo init 你的目录名
```
比如``hexo init blog``，那么就会在你的当前目录下创建一个blog文件夹，里面包含了你的博客所需要的文件<br>
接下来进入目录
```
cd blog
```
然后安装所需依赖
```
hexo install
```
一个Hexo的基本目录就安装完成了


