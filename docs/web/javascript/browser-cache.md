# 前端本地缓存概况之浏览器缓存策略

## 引子

一直以来，“前端性能优化”都是前端程序员在业务开发过程中不得不考虑的一个点。前端同学也一直寄希望于服务器更大的吞吐量、更密集的 cdn 节点；更寄希望于浏览用户使用更优秀的浏览器及更大的带宽……然而随着上述几种情况一一被落实时，前端性能仍然没有达到一个让人满意的结果……

此过程中，前端人就自身情况也进行了多种尝试，其中前端本地缓存可以说是性能优化中简单高效的一种方式，该方式缩短了网页请求资源的时长，此外缓存文件可以复用，则进一步减少了网络请求次数与实践，提高了页面加载效率。

## 前言

首先我们要明确一点：浏览器和服务器进行通讯属于“请求/应答式”，简短链路为：发起请求-服务器响应请求-获取资源。浏览器缓存就是把一个已经请求过的 Web 资源（如 html 页面，图片，js，数据等）储存在本地(内存或者硬盘)。当下一次请求要发出的时候，如果是相同的 URL，浏览器会根据缓存机制决定是直接使用先前存储的资源，还是向源服务器再次发送请求。同时而并不是所有请求都需要存储于本地（比如数据接口），那么既然我们要在浏览器层缓存特定资源，应该怎样进行判断或者约定？

## 缓存分类

### 1、强缓存

强缓存不会向服务器发送请求，直接从缓存中读取资源，在浏览器控制台的 Network 选项中可以看到该请求返回 200 的状态码，并且 Size 显示 from disk cache 或 from memory cache 等。

强缓存可以通过设置两种 HTTP Header 实现：Expires 和 Cache-Control。

#### memory cache（内存）

内存缓存，主要包含页面中已经获取到的资源，比如页面的脚本文件、样式文件、图片等，内存的读取速度要比磁盘快。该缓存属于**会话级别**，一但会话结束，则缓存资源被释放。此外内存容量所限，该缓存更多存储小体积的请求资源。

![](https://shenggao.oss-cn-beijing.aliyuncs.com/blog/2020/07/09/15942217205747.jpg)

该缓存主要关注当前会话中第一次请求返回中，response-headers 中的 cache-control 和 Expires 两个字段情况，一经识别计算通过，则资源将被存储于内存中。（一般 cdn 都会配置该策略）

1 Expires

缓存过期时间，用来**指定资源到期的时间，是服务器端的具体的时间点**。也就是：Expires=max-age + 到期时间（该到期时间为绝对时间）。Expires 是 Web 服务器响应消息头字段，在响应 http 请求时告诉浏览器在过期时间前，浏览器可以直接从浏览器缓存取数据，而无需再次请求。

    Expires 是 HTTP/1.0 的产物，受限于本地时间，如果修改了本地时间，可能会造成缓存失效。

2 Cache-Control

在 HTTP/1.1 中，Cache-Control 是最重要的规则，主要用于控制网页缓存。比如当 Cache-Control:max-age=300 时，则代表在这个请求正确返回时间（浏览器也会记录下来）的 300 秒内再次请求资源，就会命中强缓存。

**备注**：

1. Expires 和 Cache-Control 的区别在于 Expires 是 http1.0 的产物，Cache-Control 是 http1.1 的产物，两者同时存在的话，Cache-Control 优先级高于 Expires；Expires 主要是用来兼容 HTTP1.0，已经过时且存在弊端：

2. Expires 时间根据本地时间而来，如果改变本地的时间。有可能会使当前的 Expires 缓存失效。 强缓存判断是否缓存的依据来自于是否超出某个时间或者某个时间段，而不关心服务器端文件是否已经更新，这可能会导致加载文件不是服务器端最新的内容。

#### disk cache（硬盘）

其具体缓存机制与 memory cache 一致，只是可以存储的数据量比较大，但是相对来说读取略慢，比较内存缓存来说，硬盘缓存的优点主要体现在时效性上和容量上。硬盘缓存中存入的数据也是根据 http header 中的字段判定的。哪些资源可以进行存储，哪些不进行存储。

![](https://shenggao.oss-cn-beijing.aliyuncs.com/blog/2020/07/09/15942217704692.jpg)

disk cache 不同于 memory cache，disk cache 的资源是从磁盘当中取出的，也是在已经在之前的某个时间加载过该资源，不会请求服务器，但是此资源不会随着该页面的关闭而释放掉，因为是存在硬盘当中的，下次打开仍会 from disk cache。

那么，问题来了，既然 memory cache 和 disk cache 机制一致，那么哪些资源会放在内存当中，哪些资源浏览器会放在磁盘上呢？市面上不同浏览器有不同策略机制，以下以 Chrome 浏览器采取的策略简单描述一下：

| 状态 | 类型              | 说明                                                                 |
| ---- | ----------------- | -------------------------------------------------------------------- |
| 200  | form memory cache | 不请求网络资源，资源在内存当中，一般脚本、字体会存在内存当中         |
| 200  | form disk ceche   | 不请求网络资源，在磁盘当中，一般非脚本会存在内存当中，如 css、图片等 |

### 2、协商缓存(对比缓存)

协商缓存就是**强制缓存失效**后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程，需要强调的是，这个过程是**需要发出请求的**。

**备注**
强制缓存优先于协商缓存进行，若强制缓存(Expires 和 Cache-Control)生效则直接使用缓存，若不生效则进行协商缓存(Last-Modified / If-Modified-Since 和 Etag / If-None-Match)，协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，返回 200，重新返回资源和缓存标识，再存入浏览器缓存中；生效则返回 304，继续使用缓存。

#### 304

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/84653/1558062783657-92e4fc45-38a8-4c6e-84c0-1a7a890ef07c.png#align=left&display=inline&height=407&name=image.png&originHeight=407&originWidth=800&size=106021&status=done&width=800)

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/84653/1558439341166-a01890a7-d933-4292-b1b1-2536d27ef52c.png#align=left&display=inline&height=900&name=image.png&originHeight=900&originWidth=1221&size=85640&status=done&width=1221)

文件有更新，协商缓存失效，返回 200 及相关数据资源

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/84653/1558439296918-b969ce9c-32a1-4500-bb77-ae0496146822.png#align=left&display=inline&height=901&name=image.png&originHeight=901&originWidth=1225&size=97526&status=done&width=1225)

文件未更新，协商缓存生效，返回 304 及空响应体，浏览器直接读取缓存资源

如图所示，http 请求携带的缓存标识可以有两个，分别是 Last-modified 和 Etag，接下来我们慢慢说一说这两个。

**Last-modified 和 if-Modified-since**

Last-modified——最后的修改时间，根据比对修改时间可以确定在这一段时间里资源是否进行了修改。

最小颗粒为 S，这颗粒度也就暴露了这个属性的弊端，如果在一秒以内修改多次，则数据不会更新。

浏览器第一次请求的时候，响应资源的 header 中添加 last-modified，数值为资源在服务器的最后修改时间。浏览器下一次请求的时候，检测到先前返回 header 中 有“last-modified”属性，则请求上行时 header 中会添加“if-modified-since”属性，值为与“last-modified”一致。

服务器再次收到这个资源请求，会根据“If-Modified-Since”中的值与服务器中这个资源的最后修改时间对比，如果两个值相等，返回 304 和空的响应体，直接约定从浏览器缓存中读取；如果“If-Modified-Since”的时间小于服务器中这个资源的最后修改时间，说明文件有更新，于是返回新的资源文件和 200。

当然这个“last-modified”是 http1.0 年代的产物，存在着重大弊端，因为 Last-Modified 只能以秒计时，如果在同一个秒时间内修改了文件，那么此时二次请求服务端，服务端会认为资源未变更，进而返回 304，造成资源错误。

**Etag 和 if-none-macth**

前面说到了“Last-modified 和 if-Modified-since”组合的弊端，于是在后续的 http1.1 版本，引入了“Etag 和 if-none-macth”组合。

Etag 是服务器响应请求时，返回当前资源文件的一个唯一标识，一般是一个 hash 值，只要资源有变化，Etag 就会重新生成。

浏览器在下一次加载资源向服务器发送请求时，会将上一次返回的 Etag 值放到请求上行的 header 中“If-None-Match”属性里，服务器只需要比较客户端传来的 header-“If-None-Match”值跟自己服务器上该资源的 Etag 是否一致，就能直接判断资源相对客户端缓存而言是否有修改。如果服务器发现 Etag 匹配不上，那么直接返回状态码 200 及新资源（当然也包括了新的 Etag）；如果匹配是一致的，则直接返回 304 和空的响应体，直接约定从浏览器缓存中读取。这里就避免了“last-modified”的秒级误差问题。

至此，我们已经介绍了 3 种缓存：memory cache、disk cache、304，那么我们下面用一张流线图描述下请求及缓存过程：

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/84653/1558489929900-b9afc3e5-c183-4622-93ca-a2c2a49b7a17.png#align=left&display=inline&height=808&name=image.png&originHeight=808&originWidth=1258&size=97873&status=done&width=1258)

### 测试代码

```javascript
const http = require("http"); //node自带http server处理模块
const url = require("url"); //node自带路url理模块
const fs = require("fs"); //node自带文件处理模块
const path = require("path"); //node自带路径处理模块
const crypto = require("crypto"); //node自带通用加密/哈希算法模块
const PORT = 8088; //server 端口号
const mime = {
  css: "text/css",
  gif: "image/gif",
  html: "text/html",
  ico: "image/x-icon",
  jpeg: "image/jpeg",
  jpg: "image/jpeg",
  js: "text/javascript",
  json: "application/json",
  pdf: "application/pdf",
  png: "image/png",
  svg: "image/svg+xml",
  swf: "application/x-shockwave-flash",
  tiff: "image/tiff",
  txt: "text/plain",
  wav: "audio/x-wav",
  wma: "audio/x-ms-wma",
  wmv: "video/x-ms-wmv",
  xml: "text/xml",
}; //单体-response文件类型

/** 生成hash值 */
const getHash = function (str) {
  const shasum = crypto.createHash("sha1");
  return shasum.update(str).digest("base64");
};
const server = new http.Server(); //创建server
/** 监听server 请求 */
server.on("request", function (req, res) {
  const pathname = url.parse(req.url).pathname;
  const realPath = path.join(__dirname, pathname);
  let ext = path.extname(pathname);
  ext = ext ? ext.slice(1) : "unknown"; //获取请求文件后缀
  const contentType = mime[ext] || "text/plain";
  //判断文件状态，当然也可以用fs.exists()方法，但此处需要读取文件修改时间，必须使用stat
  fs.stat(realPath, (error, stat) => {
    console.log("文件请求：" + pathname);
    if (!error) {
      //根据路径读取server端资源
      fs.readFile(realPath, "binary", (err, file) => {
        if (err) {
          res.writeHead(500, {
            "Content-Type": "text/plain",
          });
          res.end(JSON.stringify(err));
          console.log("500错误：" + pathname);
        } else {
          const hash = getHash(file); //require("crypto").createHash('sha1').update(pathname).digest('base64');
          const lastModified = stat.mtime.toUTCString(); //server端对应文件最后修改日期
          if (
            req.headers["if-none-match"] === hash ||
            req.headers["if-modified-since"] === lastModified
          ) {
            res.writeHead(304); //Etag或Last-modified一致，直接返回304及空体
            res.end();
            return;
          }
          res.writeHead(200, {
            "Content-Type": contentType,
            // 'Cache-Control': 'max-age=1000',//设定缓存1000秒
            // 'Expires': new Date('Fri May 27 2020 14:53:17 GMT+0800').toUTCString(),//设定具体缓存到期时间
            // "Last-Modified": lastModified,//声明最后文件修改时间
            Etag: hash, //声明文件哈希值
          });
          res.write(file, "binary");
          res.end();
        }
      });
    } else {
      res.writeHead(404, {
        "Content-Type": "text/plain",
      });
      res.write(
        "[404] This request URL [" +
          pathname +
          "]" +
          "  was not found on this server. [404]"
      );
      res.end();
      console.log("404错误：" + pathname);
    }
  });
});
server.listen(PORT);
console.log("Server running at http://127.0.0.1:" + PORT + "/");
```
