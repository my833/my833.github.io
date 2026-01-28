+++
date = '2026-01-28T16:12:45+08:00'
draft = false
title = 'introduction to Hugo'
tags = [
  'Hugo'
]
+++

## 安装`hugo`
通过系统包管理器或直接从github下载

## 创建站点及应用主题
```bash
hugo new site myblog
cd myblog
cd themes
git clone https://git.andy.sb/nojs.git
```
编辑`hugo.toml`，添加主题及菜单：
```toml
theme = "nojs"

[menus]
  [[menus.main]]
    name = 'Home'
    pageRef = '/'
    weight = 1

  [[menus.main]]
    name = 'Posts'
    pageRef = '/posts'
    weight = 2

  [[menus.main]]
    name = 'Tags'
    pageRef = '/tags'
    weight = 3
```

修改字体：  
1. 修改`themes/nojs/assets/css/main.css`，增加新字体名称:  
  把`font-family: sans-serif;`改为`font-family: "LXGW WenKai Screen", sans-serif;`；
2. 修改`themes/nojs/layouts/_partials/head.html`，增加样式:  
  添加`<link rel="stylesheet" href="https://npm.elemecdn.com/lxgw-wenkai-screen-webfont/style.css" />`。


## Github Pages & Actions
正常建立repo，点击`Actions`，搜索`hugo`，点击`Configure`直接保存。

## 其他
其他主题: [yingyang](https://github.com/joway/hugo-theme-yinyang).
