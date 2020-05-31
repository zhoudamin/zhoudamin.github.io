---
title: Hexo Notes
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-04-23 23:07:42
categories: Notes
tags: Hexo
---
遇到的小问题

<!--more-->
# 关于设置页宽
没错，这就是我上面提到的那个主题，这个主题有三种外观，其中我现在使用的是 Pisces Scheme ，但还是做了一些修改，因为原来那个宽度太小了，不适合展示代码块，也不太美观。修改方法如下：
Pisces 的布局定义在 source/css/_schemes/Picses/_layout.styl 中，打开文件并在最后添加以下 css
引用自  http://www.aidansu.com/2017/github-pages-build-blog/
```css
.header{
    width: 80%;
    +tablet() {
        width: 100%;
    }
    +mobile() {
        width: 100%;
    }
}
.container .main-inner {
    width: 80%;
    +tablet() {
        width: 100%;
    }
    +mobile() {
        width: 100%;
    }
}
.content-wrap {
    width: calc(100% - 260px);
    +tablet() {
        width: 100%;
    }
    +mobile() {
        width: 100%;
    }
}
```
