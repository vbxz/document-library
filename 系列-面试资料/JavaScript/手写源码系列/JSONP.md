JSONP
===

> Create by **jsliang** on **2020-09-29 00:43:41**  
> Recently revised in **2020-10-12 22:03:35**

<!-- 目录开始 -->
## <a name="chapter-one" id="chapter-one"></a>一 目录

**不折腾的前端，和咸鱼有什么区别**

| 目录 |
| --- |
| [一 目录](#chapter-one) |
| <a name="catalog-chapter-two" id="catalog-chapter-two"></a>[二 前言](#chapter-two) |
| <a name="catalog-chapter-three" id="catalog-chapter-three"></a>[三 基本原理、执行过程和优缺点](#chapter-three) |
| <a name="catalog-chapter-four" id="catalog-chapter-four"></a>[四 基本实现](#chapter-four) |
| &emsp;[4.1 步骤一：前端部分](#chapter-four-one) |
| &emsp;[4.2 步骤二：后端部分](#chapter-four-two) |
| &emsp;[4.3 步骤三：前端部分](#chapter-four-three) |
| <a name="catalog-chapter-five" id="catalog-chapter-five"></a>[五 手写 JSONP](#chapter-five) |
| &emsp;[5.1 封装 JSONP](#chapter-five-one) |
| &emsp;[5.2 简单实现](#chapter-five-two) |
| &emsp;[5.3 完善版本](#chapter-five-three) |
<!-- 目录结束 -->

## <a name="chapter-two" id="chapter-two"></a>二 前言

> [返回目录](#chapter-one)

## <a name="chapter-three" id="chapter-three"></a>三 基本原理、执行过程和优缺点

> [返回目录](#chapter-one)

* **基本原理**：利用 `script` 标签的 `src` 没有跨域限制来完成实现。

* **执行过程**：

1. 前端定义一个解析函数：`jsonpCallBack = function (res) {}`
2. 通过 `params` 的形式包装 `script` 标签的请求参数，并且声明执行函数，如：`cb = jsonpCallBack`
3. 后端获取前端声明的执行函数（`jsonpCallBack`），并且带上参数且调用函数的方式传递给前端
4. 前端再 `script` 标签返回资源的时候就会执行 `jsonpCallBack` 并通过回调函数的方式拿到数据。

* **优缺点**：

1. 【缺点】只能进行 `GET` 请求
2. 【优点】兼容性好，在一些古老的浏览器中都可以运行。

## <a name="chapter-four" id="chapter-four"></a>四 基本实现

> [返回目录](#chapter-one)

### <a name="chapter-four-one" id="chapter-four-one"></a>4.1 步骤一：前端部分

> [返回目录](#chapter-one)

前端代码：

> index.html

```html
<script type='text/javascript'>
  window.jsonpCallBack = function(res) {
    console.log(res);
  }
</script>
<script src='http://localhost:8080/api/jsonp?id=1&cb=jsonpCallBack' type='text/javascript'></script>
```

1. 创建一个 `jsonpCallBack` 函数，但是还没有被调用。
2. 加载 `src` 中的资源，调用 `localhost:8080` 端口的 API：`api/jsonp`，传递的参数是 `id = 1` 以及 `cb = jsonpCallBack`

### <a name="chapter-four-two" id="chapter-four-two"></a>4.2 步骤二：后端部分

> [返回目录](#chapter-one)

后端代码：

```js
const Koa = require('koa');
const app = new Koa();
const items = [{ id: 1, title: 'title1' }, { id: 2, title: 'title2' }]

app.use(async (ctx, next) => {
  if (ctx.path === '/api/jsonp') {
    const { cb, id } = ctx.query;
    const title = items.find(item => item.id == id)['title']
    ctx.body = `${cb}(${JSON.stringify({title})})`;
    return;
  }
})
console.log('listen 8080...')
app.listen(8080);
```

1. 解析 `ctx.query`，将 `id` 和 `cb` 获取到。
2. 查找 `title`
3. 返回一个字符串，该字符串调用 `cb` 方法，并将 `title` 转成字符串，返回到内容 `ctx.body` 中。

### <a name="chapter-four-three" id="chapter-four-three"></a>4.3 步骤三：前端部分

> [返回目录](#chapter-one)

这时候前端接收到 Node.js 的返回，直接调用了 `cb(title)`，方法，最终执行了它的回调，从而执行：

```js
window.jsonpCallBack = function(res) {
  console.log(res);
}
```

输出：`{ title: 'title1' }`

## <a name="chapter-five" id="chapter-five"></a>五 手写 JSONP

> [返回目录](#chapter-one)

### <a name="chapter-five-one" id="chapter-five-one"></a>5.1 封装 JSONP

> [返回目录](#chapter-one)

定义一个 `JSONP` 方法，接收 4 个参数：

* `url`：`api` 接口
* `params`：`api` 接口参数
* `callbackKey`：与后端约定回调函数用哪个字段
* `callback`：拿到数据之后前端执行的回调函数

```js
JSONP({
  url: 'https://www.baidu.com/s',
  params: { wd: 'jsliang' },
  callBackkey: 'cb',
  callback(res) {
    console.log(res);
  }
})
```

### <a name="chapter-five-two" id="chapter-five-two"></a>5.2 简单实现

> [返回目录](#chapter-one)

我们看看简单实现：

```js
const JSONP = ({
  url,
  params = {},
  callBackkey = 'cb',
  callback,
}) => {
  params[callBackkey] = callBackkey;

  window[callBackkey] = callback;

  const newParam = Object.keys(params).map((key) => `${key}=${params[key]}`).join('&');

  const script = document.createElement('script');
  script.setAttribute('src', `${url}?${newParam}`);
  document.body.appendChild(script);
}

JSONP({
  url: 'https://www.baidu.com/s',
  params: { wd: 'jsliang' },
  callBackkey: 'cb',
  callback(res) {
    console.log(res);
  }
})
```

### <a name="chapter-five-three" id="chapter-five-three"></a>5.3 完善版本

> [返回目录](#chapter-one)

优化多次调用时候的一个问题：

```js
function JSONP({
  url,
  params = {},
  callbackKey = "cb",
  callback,
}) {
  // 定义本地的唯一 callBackId，防止多次调用的时候出问题
  JSONP.callBackId = JSONP.callBackId || 1; // 默认为 1

  // 拿到这个 id
  const callBackId = JSONP.callBackId;

  // 将要执行的回调假如到 JSONP 对象中，避免污染 window
  JSONP.callbacks = JSONP.callbacks || [];
  JSONP.callbacks[callBackId] = callback;

  // 把这个名称加入到参数中：`cb = JSONP.callbacks[1]`
  params[callbackKey] = `JSONP.callbacks[${callBackId}]`;

  // 组合 params：'id=1&cb=JSONP.callbacks[1]'
  const paramString = Object.keys(params).map((key) => `${key}=${params[key]}`);

  // 动态创建 script 标签
  const script = document.createElement("script");
  script.setAttribute("src", `${url}?${paramString}`);
  document.body.appendChild(script);

  // id 自增，保证唯一
  JSONP.callBackId++;
}

JSONP({
  url: "http://localhost:8080/api/jsonp",
  params: { id: 1 },
  callbackKey: "cb",
  callback(res) {
    console.log(res);
  },
});
JSONP({
  url: "http://localhost:8080/api/jsonp",
  params: { id: 2 },
  callbackKey: "cb",
  callback(res) {
    console.log(res);
  },
});
```

---

> <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">jsliang 的文档库</span> 由 <a xmlns:cc="http://creativecommons.org/ns#" href="https://github.com/LiangJunrong/document-library" property="cc:attributionName" rel="cc:attributionURL">梁峻荣</a> 采用 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享 署名-非商业性使用-相同方式共享 4.0 国际 许可协议</a>进行许可。<br />基于<a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/LiangJunrong/document-library" rel="dct:source">https://github.com/LiangJunrong/document-library</a>上的作品创作。<br />本许可协议授权之外的使用权限可以从 <a xmlns:cc="http://creativecommons.org/ns#" href="https://creativecommons.org/licenses/by-nc-sa/2.5/cn/" rel="cc:morePermissions">https://creativecommons.org/licenses/by-nc-sa/2.5/cn/</a> 处获得。