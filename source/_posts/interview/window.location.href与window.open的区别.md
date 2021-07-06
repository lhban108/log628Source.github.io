---
layout: JS
title: window.location.href与window.open的区别
date: 2019-08-09 14:33:28
tags: [interview, js]
---

## 一、比较常用的JS跳转页面和打开新窗口的方法

### 1、替换当前页 （重新定位当前页）

```javascript
window.location.href = "https://www.xxx.com"; // 跳转到新的域名
window.location.href = `/dashboard#/setting?type=1&userId=123`; // 在当前域名下跳转到新的子页面

window.location.href = "https://www.xxx.com"; // 跳转到新的域名
window.location.href = `/dashboard#/setting?type=1&userId=123`; // 在当前域名下跳转到新的子页面
```

### 2、打开新窗口

```javascript
window.open("https://www.xxx.com"); // 跳转到新的域名
window.history.back(-1); // 返回到上一页（在当前窗口 ）
```

## 二、`window.location.href` 与 `window.open()` 的区别

- 区别一  
　　`window.location`是`window`对象的**属性**  
　　`window.open()`是`window`对象的**方法**

- 区别二  
　　`window.location.href`是用新的域名**替换当前页**， 也就是重新定位当前页  
　　`window.open()`是用来**打开一个新窗口**的函数！

- 区别三
　　`window.open()`可能会被浏览器拦截  
　　`window.location.href`不会被窗口拦截

- `window.location.href` 和 `document.location.href`的区别：
　　`window.location.href` 和 `document.location.href`都可以对当前窗口进行重定向。（尽管 `Document.location` 是一个只读的 `Location` 对象，但是也能够赋给它一个 `DOMString`）
　　当服务器未发生重定向时, 两者是相同的。
　　但是当服务器发生了重定向,就不一样了:
　　- document.location包含的是已经装载的URL
　　- ocation.href包含的则是原始请求的文档的URL

## 三、`window.location.href`怎么跳转新窗口

`window.location.href`是在当前窗口进行覆盖，那怎么跳转到新窗口呢？

```javascript
let tempwindow = window.open('_blank');
tempwindow.location = 'https://www.xxx.com'; // 可以打开新的地址
// tempwindow.location = '/dashboard#/setting?type=1&userId=123'; // 也可以打开原有地址的子页面
```

>参考资料：
> [window.location.href和window.open的几种用法和区别](https://www.cnblogs.com/Qian123/p/5345298.html)
> [window.location.href怎么跳转新窗口](https://www.imooc.com/wenda/detail/323004)
