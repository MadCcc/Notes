# 如何从研发的角度提升网站SEO质量

## 前言

Search Engine Optimization: The process of improving the **quality** and **quantity** of website traffic to a website or a web page from **search engines**.

SEO对于一个想要在搜索引擎上获得较高排名的网站来说是必不可少的，那么如何提高网站的SEO质量呢？本文将会挑几点重要并且有效的详细讲解。

先上一些参考资料，详细的如何优化网站的SEO可以在这些资料中找到
- [Google Search Central](https://developers.google.com/search/docs)
- [Moz](https://moz.com/beginners-guide-to-seo)

## 技术选型
SEO是针对搜索引擎的优化，而搜索引擎在爬取站点的时候并不是以浏览器的方式，而是类似于curl的方式去爬取，所以我们必须要保证搜索引擎在访问我们的站点时能够更加快速有效的获取有效信息。

这里不推荐使用纯SPA框架，如简单的create-react-app或者vue-cli创建的项目。这些项目都是客户端渲染的（CSR，Client Side Rendering），这也就意味着你去curl这个网页只会得到一段毫无任何信息的HTML，所有的页面内容都是在本地（浏览器）运行js代码之后生成的。

如果你的站点都是静态内容，我会推荐使用一些支持SSG（Static Site Generation）的框架，比如[Gasbyjs](https://www.gatsbyjs.com/)，这类框架可以在构建时将你的网站直接构建为一个完整的HTML，并部署到你的云平台上。这时候搜索引擎爬取你的站点时就会拿到有用的信息了。

如果你的站东动态内容较多，推荐使用支持SSR（Server Side Rendering）的框架，比如[Nextjs](https://nextjs.org/)或者[Nuxtjs](https://nuxtjs.org/)，它们分别基于React和Vue框架，并且也同时支持SSG构建。SSR会在用户访问站点时在后台生成完整的HTML，所以可以实时地获取一些动态数据。

## Robots.txt
robots.txt文件可以有效的管理流向你的站点的流量，搜索引擎都会遵循站点下的robots.txt文件里制定的规则。robots.txt文件中可以指定某个搜索引擎的bot（如googlebot）是否可以访问你的站点下的某些路径，如果明令禁止了，搜索引擎将不会爬取你规定的站点路径。默认情况下所有的站点路径都可以被爬取。下面是一个例子：

```
# Group 1
User-agent: Googlebot
Disallow: /nogooglebot/

# Group 2
User-agent: *
Allow: /

Sitemap: http://www.example.com/sitemap.xml
```
这个简单的robots.txt文件禁止了googlebot爬取`/nogooglebot/`路径下的所有页面。同时还指定了这个站点下的sitemap的地址，可供搜索引擎获取sitemap

## Sitemap
sitemap通常是一个xml文件，里面你可以提供所有你希望搜索引擎爬取的网页/视频等静态资源。有了sitemap，搜索引擎爬取你的网站时会更有指向性。下面是一个例子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:news="http://www.google.com/schemas/sitemap-news/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml" xmlns:mobile="http://www.google.com/schemas/sitemap-mobile/1.0" xmlns:image="http://www.google.com/schemas/sitemap-image/1.1" xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">
  <url>
    <loc>http://example.com</loc>
    <changefreq>weekly</changefreq>
    <priority>1</priority>
    <lastmod>2021-09-29T09:14:23.422Z</lastmod>
  </url>
  <url>
    <loc>http://example.com/about</loc>
    <changefreq>weekly</changefreq>
    <priority>1</priority>
    <lastmod>2021-09-29T09:14:23.422Z</lastmod>
  </url>
</urlset>
```

上面的sitemap就提供了两个页面给搜索引擎，同时也可以指定一下其他的属性，比如`changefreq`代表这个页面的更新频率，`priority`代表这个页面的权重，`lastmod`代表这个页面的上次修改时间等等。

sitemap数量不宜过多，2000～3000条会是一个比较合适的数量。如果数量太多的话我们可以把sitemap下分到子路径中，如我们可以同时提供`http://example.com/sitemap.xml`与`http://example.com/about/sitemap.xml`。

## Meta信息

Meta信息包括网站的title/description/keywords，其中google已经明确表示不会对keywords收集信息，所以可以忽略。一个SEO友好的站点，所有页面都需要有精准的meta信息，用于存放一些关键词与这个页面的主要内容。

我们需要把这些meta信息放在网页的`<head>`标签中，例如vue的官网首页：

```
<title>Vue.js</title>
<meta name="description" content="Vue.js - The Progressive JavaScript Framework">
```

## Internal links
Internal links，内链，特指那些可以链接到你的站点内其他页面的链接（a标签）。这些链接会被搜索引擎记录并爬取，所以非常有利于构建站点的拓扑结构。因此在网站中加入大量内链是可以提升网站整体的SEO质量的。

值得一提的是，现在广泛使用的前端框架如React和Vue都是有客户端路由的，它们也提供了两种跳转方式：命令式与声明式，以vue为例，`vue-router`提供了`push`等方法供用户在js代码中实现页面跳转，这些就是命令式，他们并不会在你的网页元素上留下a标签，所以这种方式对于SEO式不友好的。因此我们要尽量多地使用声明式的跳转，也就是利用框架提供的跳转组件，它们会为我们的网页实现SPA式的快速跳转，并且也留下带有信息的a标签在网页元素中。

## Canonical Url

Canonical Url可以翻译为规范网址，常用于“合并同类项”。有以下几种场景我们可以用到规范网址：
1. 明确指定你的网页在搜索引擎的搜索结果中展现的网址
2. 合并相似的或者重复的但具有不同网址的网页
3. 对于同一个网页降低流量跟踪的难度
4. 方便管理联合内容
5. 避免搜索引擎花太多时间爬取重复的网页

我们同样需要在`head`标签中指定规范网址，例如：

```html
<link rel="canonical" href="https://example.com">
```

## URL结构

URL就是一个网页的地址，那么对于一个SEO有好的网站，其内部所有网页都需要用URL分隔开来，以保证搜索引擎能够爬取到站点中的任何一个页面。

一个好的URL结构需要具备以下几条素质：

### 有意义的命名

也就是说就像为变量取名一样，你的URL中的每一段也必须是有意义的

```
❌  https://example.com/aaaaaaa
⭕️  https://example.com/about
```

### 反映网站结构

你的url同样可以反映你的站点的结构，所以你可以像建立文件夹一样为你的页面划分路径，比如：

```
https://example.com/article/how-to-use-github
https://example.com/article/my-first-repo-in-github
```

###  具备关键词

url中可以像title/description一样塞入一些关键词，但注意要保证url的简洁

```
https://example.com/awards
https://example.com/price
```

### 易读性

减少URL中查询参数/数字/符号的使用，提高URL的易读性

```
❌  https://github.com/MadCcc/Notes
⭕️  https://example.com/article?id=123456678
```

## 图片的Alt属性

对于搜索引擎的bot来说，图片只是一个元素而已，所以图片的alt属性的地位就变得很高，它代表了这个图片的意义，所以我们有必要在所有`img`元素上添加`alt`属性，并附上符合上下文的描述文字

## Tab组件于Dropdown组件

构建页面的时候我们有时候会用到这两个组件，它们或者其他一些类似的组件都有一个共性：隐藏了一部分元素。因此隐藏的部分我们必须尤其注意，比如一些重要的信息如内链等，就会隐藏在这些组件中。

对于这种会隐藏信息的组件，我们在实现它们或者在选用其他社区提供的插件时，需要让他们能够以`display: none`的形式被隐藏，而非利用js的动态隐藏与生成，这样它们至少还存在于HTML中，可以被搜索引擎发现。

Tab组件我们则可以动态处理，利用SPA框架提供的feature，我们可以为每一个tab指定一个url，这样也可以让每一个tab下的内容被搜索引擎发现。

## 网站性能优化

网站性能同样是衡量SEO的重要标准，但这里就不展开讲了，我们可以在这个网站中了解更多：https://developers.google.com/speed/pagespeed/insights/

网页性能有三个比较重要的指标：Largest Contentful Paint (LCP)、First Input Delay (FID)、Cumulative Layout Shift (CLS)，上面这个网站可以帮我们测量，这些可以作为我们优化网站的一个根据

## 总结

做SEO优化是一个非常零碎的工作，大部分答案其实都藏在google以及其他搜索引擎提供的文档中，属于是那种肯花时间阅读就一定能做好的活儿。但是从SEO优化的一些细碎标准中我们仍人可以学到很多平常会被我们忽略的小细节，所以**细节**在前端还是非常重要的。

