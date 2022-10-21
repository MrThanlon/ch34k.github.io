---
title: 如何优雅地调用web接口(api.js)
tags:
  - JavaScript
  - Web
categories:
  - Web
  - JavaScript
  - Proxy
abbrlink: 544
date: 2020-06-22 15:27:05
---

在写前端项目的时候总免不了要调用后端提供的接口，比如POST `/user/login`，比较传统的方法就是使用XHR。不过，ES6引入了Promise，ES7引入了async/await，使得我们可以更优雅地完成异步操作，于是[axios](https://github.com/axios/axios)隆重登场（其实我个人并没有用过axios）。

<!--more-->

主要是受到张大佬的启发，通过

```javascript
await api.user.login()
```

的方式将实际的Web接口封装成一个对象，但是他那个版本做得还不够好，需要事先定义出所有允许调用的接口，然后再处理成一个`api`对象。

我改进了一下，使用Proxy对象，可以不需要提前定义接口，直接调用即可，这就是现在的[api.js](https://gist.github.com/MrThanlon/232ac1eeab73d318a5067ceecbec437a)。

<script src="https://gist.github.com/MrThanlon/232ac1eeab73d318a5067ceecbec437a.js"></script>

说实话，这里的Proxy+递归用的非常花俏，我感觉自己也是抽风了才写出来的。

嘛，总之用法就很简单了，看demo.js的操作。调用的函数可以带参数，如果是一般`Object`则会以`Content-Type: application/json`方式发送，如果是`FormData`则会以`Content-Type: multipart/form-data`方式发送，超级舒适的。

## <del>严重Bug</del>

- <del>请求的链接中不能含有`/apply/`或`/apply`，否则导致报错，原因是`apply`是关键字，关于如何规避我还没想到一个合理的写法。</del>