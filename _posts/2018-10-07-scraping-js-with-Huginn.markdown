---
layout:       post
title:        "RSS 进阶篇：Huginn - 真·为任意网页定制 RSS 源（PhantomJs 抓取）"
subtitle:     ""
date:         2018-10-7
author:       "Benson"
header-img:   img/post-bg-20180108.jpg
header-mask:  0.3
catalog:      true
tags:
    - Huginn
    - RSS
    - PhantomJs Cloud
---
# RSS 进阶篇：Huginn - 真·为任意网页定制 RSS 源（PhantomJs 抓取）

烧制网页RSS源主要有**FEED43**和**Huginn**两种方法。
1. FEED43：简单免费，六小时抓取一次，每次抓取20条静态页面。使用攻略-[RSS 入门篇：FEED43&FeedEx-为静态网页定制 RSS 源](https://zhuanlan.zhihu.com/p/26511654)
2. Huginn：自由度高，可设定**抓取频率、内容结构、js结果、输出样式**等；需要搭建服务器，学习Huginn抓取规则


### **Huginn 准备工作**：

1. 准备一台 Debian/Ubuntu 环境的服务器
2. 按[Qi大的攻略](https://wzfou.com/huginn/)搭建Huginn，也可以直接看[Huginn 官方搭建攻略](https://github.com/huginn/huginn/blob/master/doc/manual/installation.md)

准备工作完成后，我们已经可以使用 Huginn 抓取页面了。但很多网站都是用 JS 加载动态内容，需要通过 **PhantomJs Cloud** 抓取页面 JS 缓存。

————

### Huginn + PhantomJs Cloud 全网页抓取

#### 一、Phantom Js Cloud API key 获取
注册 [PhantomJs Cloud](https://phantomjscloud.com/) ,然后将 API key 保存在 Huginn 的 Credentials 中。
![](http://tc.seoipo.com/20181006010447.png)

新建 Huginn 任务组 Scenario 「国内应急新闻」，抓取链接 http://www.cneb.gov.cn/guoneinews/

![](http://tc.seoipo.com/20181008131549.png)

####  二、Phantom Js Cloud Agent 抓取页面缓存

   *Name: 国内应急新闻 #1 获取 JS 缓存*
   *Schedule: Every 1h*
![](http://tc.seoipo.com/20181008111704.png)

####  三、WebsiteAgent 获取页面详情
   *Name: 国内应急新闻 #2 抓取全页*
   *Schedule: Every 1h*
![](http://tc.seoipo.com/20181008112658.png)

####  四、css path 路径获取
1. 使用火狐浏览器打开抓取页面
2. 按下`F12`, 然后点击 *Developer Tools* 左上角的*检查指针*
![](http://tc.seoipo.com/20181008114911.png)
3. 选中要抓取的部分
![](http://tc.seoipo.com/20181008113925.png)
4. 回到 *Developer Tools* 窗口，右键选中的蓝色部分，获取 css path、Xpath。这里以 css path 为例。
![](http://tc.seoipo.com/20181008114207.png)
5. 处理 css path 路径
```css
html body div.area.areabg1 div.area-half.right div.tabBox div.tabContents.active table tbody tr td.red a
```
css path 原始路径过长，删去不带 `.` 或 `#` 的节点（节点间以空格“ ”分割），并删去每个节点在 `.` 或 `#`前的第一个标签，得到：
```css
.area.areabg1 .area-half.right .tabBox .tabContents.active .red a
```
前半部分对节点定位无用，继续省略（比如：中国上海，省略掉中国，大家也知道上海在哪）
```css
.tabContents.active .red a
```
**非常规情况处理**：
a. 有些路径中的**节点带空格**，如`<div class="packery-item article">`,路径中的空格由`.`代替，截取为`.packery-item.article`
b. 当抓取**多种 css path 规则**时，用逗号,分割
```css
"css": ".focus-title .current a , .stress h2 a",
```

#### 五、DataOutputAgent 导出 RSS

   *Name: 国内应急新闻 #3 排序生成RSS*
   **Propagate immediately*: Yes*

![](http://tc.seoipo.com/20181008130943.png)

回到Scenarios, 点击最后一步的 Actions - Show ，复制导出的xml链接 `http://xxx.xxxxxx/users/1/web_requests/xxx/xxxx.xml`

![](http://tc.seoipo.com/20181008131059.png)

详细设置的使用文件-[百度网盘下载](https://pan.baidu.com/s/1JdsFkLN9kczR9C92tKi83A)

其他问题，查看官方说明-[PhantomJs Cloud 英文使用攻略](https://github.com/huginn/huginn/wiki/Browser-Emulation-Using-PhantomJs-Cloud)

