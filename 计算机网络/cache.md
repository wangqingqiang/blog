前端缓存可以在**请求**和**响应**两步中作用：
- 请求步骤中浏览器直接使用本地缓存的资源，省去了发送请求
- “响应”步骤需要浏览器和服务器共同配合，通过**减少响应内容（大小）**来缩短传输时间

## 分类
- Service Worker
- memory cache
- disk cache  

优先级：**Service Worker > memory cache > disk cache > 网络请求**  
如果 `Service Worker` 没能命中缓存，一般情况会使用 `fetch()` 方法继续获取资源。这时候，浏览器就去 `memory cache` 或者 `disk cache` 进行下一次找缓存的工作.注意：经过 Service Worker 的 fetch() 方法获取的资源，即便它并没有命中 Service Worker 缓存，甚至实际走了网络请求，也会标注为 from ServiceWorker

## disk cache缓存策略
### 强制缓存
只要存在缓存就用缓存，不发送请求，没有缓存再发送请求并缓存请求结果。**强制缓存直接减少了网络请求，提升最明显**
协议头有 `expires`、 `cache-control`
#### expires
```
expires: Wed, 31 Jan 2029 06:13:08 GMT
```
`HTTP 1.0`的字段，**表示缓存到期时间，是一个绝对时间**  
**缺点**：
- 时间有偏差，服务端和客户端的时间设置可能不同，这就会使缓存的失效可能并不能精确的按服务器的预期进行
#### cache-control
```
cache-control: max-age=315360000
```
由于 `expries` 的缺点，`HTTP 1.1`新增了 `cache-control` 字段，**表示缓存的最大时长，是一个相对时间，单位是秒**，cache-control常用字段：
- max-age: 缓存时长，单位秒
- no-cache: 表示必须先与服务器确认返回的响应是否发生了变化，然后才能使用该响应来满足后续对同一网址的请求。 因此，如果存在合适的验证令牌 (ETag)，no-cache 会发起往返通信来验证缓存的响应，但如果资源未发生变化，则可避免下载。
- no-store: **真正的不缓存**，所有内容都不走缓存，包括强制和对比
- public：所有的内容都可以被缓存 (包括客户端和代理服务器， 如 CDN)
- private：默认值。所有的内容只有客户端才可以缓存，代理服务器不能缓存。

**`cache-control` 的优先级高于 `expires`**，为了兼容 HTTP/1.0 和 HTTP/1.1，实际项目中两个字段我们都会设置

### 协商缓存（对比缓存）
当强制缓存失效后就需要使用协商缓存，由服务器决定缓存是否失效。  
浏览器先访问缓存，获取一个缓存标识，然后拿着这个标识请求服务器，如果缓存未失效返回 `304` 标识继续使用缓存；如果失效了就返回新的数据和缓存规则。  
我们发现 `协商缓存并不会减少HTTP请求数，但是可以减少请求响应体积，以此缩短网络传输时间，和强制缓存相比提升幅度较小`。  
协商缓存有两组字段：
`last-modified / if-modified-since` 和 `etag / if-none-match`
#### last-modified / if-modified-since
- 服务器返回`last-modified` 告知浏览器资源最后被修改时间，绝对时间
- 需要协商缓存时，浏览器将上次`last-modified` 的值写入请求头 `if-modified-since` 
- 服务器将请求头中的 `if-modified-since` 的值和服务器中最新文件的 `last-modified` 值比较，如果一致返回 `304` ，否则返回新的数据同时状态码为`200`

**缺点：**
- `Last-Modified / If-Modified-Since` 的主要缺点就是它只能精确到秒的级别，一旦在一秒的时间里出现了多次修改，那么`Last-Modified / If-Modified-Since`是无法体现的
- 如果文件是通过服务器动态生成的，那么`Last-Modified` 永远是当前请求时间，尽管文件可能没有变化，仍然起不到缓存的作用

#### etag / if-none-match
`etag / if-none-match` 的流程和 `last-modified / if-modified-since` 类似，但是**没有使用时间作为判断标准，而是文件的特殊标识(一般都是 hash 生成的)**
- 服务器返回 `etag`
- 浏览器请求资源时将 `etag` 值写入请求头 `if-none-match` 中
- 服务器判断请求头中的 `if-none-match` 和自己保存的值是否相同，相同返回`304`，不同返回 `200` 和新文件

**缺点：**
后台是多台服务器负载均衡，不同服务器生成的`etag` 值可能会不同，造成缓存失效的情况。

## 总结
请求一个资源：  
1. Service Worker
2. memory cache
3. disk cache:
   1. 如果强制缓存未失效，使用强制缓存，不发送请求，状态码 `200`
   2. 强制缓存失效，使用协商缓存，比较后确定是 `304` 还是 `200`


场景：
- js、css等静态资源使用较长时间的强缓存+协商缓存，为了便于资源更新后浏览器及时更新可以结合`hash`使用
- html页面可以只用协商缓存，`cach-control` 设置成`max-age=0`或者`no-cache`

参考：[一文读懂前端缓存](https://zhuanlan.zhihu.com/p/44789005)
