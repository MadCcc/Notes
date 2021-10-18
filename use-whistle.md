# 前言
前端开发最重要的一环就是对接后端请求，正常情况下从浏览器发出的请求可以在浏览器的开发者工具的Network一栏看到。但对于服务端渲染的前端项目来说，大部分请求都是在服务端发出的，所以并不能在浏览的Network中监控到，所以我们会采用“抓包”的形式在开发过程中针对服务端请求进行监测。
常用的抓包工具有很多，比如Charles/ProxyMan等等，利用的都是代理（Proxy）将服务端请求代理到它们的端口上，再从它们的端口上发出，因此能在这些软件上看到从服务端发出的请求。但是Charles这些客户端软件对于内存占用却相当严重，因此本文介绍一个比较轻量的抓包工具：[Whistle](https://github.com/avwo/whistle)

# 安装方法
首先你需要安装Node.JS，因为Whistle是一个npm包，需要依赖nodejs运行。安装Node.JS这里就省略了。
然后我们开始安装whistle

```
npm install -g whistle
```
也可以用中国的镜像源

```
npm install cnpm -g --registry=https://registry.npm.taobao.org
cnpm install -g whistle
```

or specify mirror install directly：
```
npm install whistle -g --registry=https://registry.npm.taobao.org
```
安装之后我们就可以直接运行whistle

```
whistle run -p 9090(或者其他你喜欢的端口号）
```
然后打开 http://localhost:9090 ，打开左侧的network一栏，就可以在这个页面监测代理到 http://localhost:9090 的所有请求了

![image](https://user-images.githubusercontent.com/27722486/134756114-04f58fda-e89e-4aa5-9c2f-a85e2c014fad.png)

# Axios配置
这时候就有同学要问了，里面怎么啥也看不到啊！别急，我们还需要将服务端请求代理到whistle运行的端口上。这里以前端常用的Axios库为例：

```js
axios.create({
  baseURL: ...(省略）,
  timeout: 10000, // timeout range
  proxy: {
    host: 'localhost',
    port: 9090,
  },
});
```
这里就是将Axios的请求代理到whistle端口上，然后所有基于这个service发出的请求就都可以在whistle可视化界面中看到啦

# 结语
服务端渲染对于前端性能以及表现来说确实是一笔相当大的提升，但维护一个服务端渲染的项目却需要花费更多的精力。抓包只是服务端渲染在开发过程中的一个小问题，还有很多其他问题需要我们去理解与攻克。但出乎我意料的是nextjs这个框架更新频率超乎我的想象，这半年来竟然已经更新了一个大版本，可见这个团队对于框架维护还是非常勤劳的（尽管还是有一些小问题存在）。所以我坚信，没有解决不了的难题，所有我们花费的时间都将会得到回报！