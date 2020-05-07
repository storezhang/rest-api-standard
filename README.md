## 目录

- [介绍](#介绍)
- [协议](#协议)
- [域名](#域名)
- [版本（Versioning）](#版本（Versioning）)
- [路径（Endpoint）](#路径（Endpoint）)
- [HTTP动词](#HTTP动词)
- [过滤信息（Filtering）](#过滤信息（Filtering）)
- [状态码（Status Codes）](#状态码（Status-Codes）)
- [错误处理（Error handling）](#错误处理（Error-handling）)
- [返回结果](#返回结果)
- [JSON格式规范](#JSON格式规范)
- [Hypermedia API](#Hypermedia-API)
- [参考资料](#参考资料)


## 介绍

网络应用程序，分为前端和后端两个部分。当前的发展趋势，就是前端设备层出不穷（手机、平板、桌面电脑、其他专用设备）。因此，必须有一种统一的机制，方便不同的前端设备与后端进行通信。这导致API构架的流行，甚至出现"API First"的设计思想。RESTful API是目前比较成熟的一套互联网应用程序的API设计理论。我以前写过一篇《理解RESTful架构》，探讨如何理解这个概念。今天，我将介绍RESTful API的设计细节，探讨如何设计一套合理、好用的API

**RESTful API的设计有很多种实现，这实现不一定是最好的，但是相对来说是我工作8年以来，最好的实现方案**


## 协议

API与用户的通信协议，**总是使用HTTPs协议**，需要注意的是，HTTPs协议是指对外协议而不是指任何时候都是HTTPs协议，现实一点说，真实的开发中，**内部使用HTTP协议，外部由网关或者Nginx转成HTTPs协议**


## 域名

应该尽量将API部署在专用域名之下，比如`https://api.example.com`，如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下`https://example.org/api/`


## 版本（Versioning）

版本体现了API现在的实现情况，在做向上向下兼容时十分有用，有两种处理方式

- 应该将API的版本号放入URL：`https://api.example.com/v1/`
- 另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。Github采用这种做法


## 路径（Endpoint）

路径又称"终点"（endpoint），表示API的具体网址。在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样

- https://api.example.com/v1/zoos
- https://api.example.com/v1/animals
- https://api.example.com/v1/employees

注意点
- 使用名称而非动词如`persons` `works` `users`等
- 使用复数形式，**禁止使用单数形式**
- 全部使用小写字母，单词之间使用中划线`-`分隔


## HTTP动词

对于资源的具体操作类型，由HTTP动词表示常用的HTTP动词有下面五个（括号里是对应的SQL命令）

- GET（SELECT）：从服务器取出资源（一项或多项）
- POST（CREATE）：在服务器新建一个资源
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
- PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）
- DELETE（DELETE）：从服务器删除资源

还有两个不常用的HTTP动词

- HEAD：获取资源的元数据
- OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的

下面是一些例子

- GET /zoos：列出所有动物园
- POST /zoos：新建一个动物园
- GET /zoos/ID：获取某个指定动物园的信息
- PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
- PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
- DELETE /zoos/ID：删除某个动物园
- GET /zoos/ID/animals：列出某个指定动物园的所有动物
- DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物


## 过滤信息（Filtering）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。下面是一些常见的参数

- ?limit=10：指定返回记录的数量
- ?offset=10：指定返回记录的开始位置。
- ?page=2&perPage=100：指定第几页，以及每页的记录数。
- ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
- ?animalTypeId=1：指定筛选条件

**参数的设计允许存在冗余**，即允许API路径和URL参数偶尔有重复。比如，GET`/zoo/ID/animals`与 GET`/animals?zooId=ID`的含义是相同的


## 分页处理

分页处理在所有的API里都存在，且强烈建议**GET`/zoos`API返回分页对象**

### 分页请求对象
```javascript
{
    // 当前页数
    "page": 1,
    // 每页对象个数
    "perPage": 20,
    // 搜索关键字
    "keyword": "",
    // 排序字段
    "sortOrder": "id DESC"
}
```

对于分页对象
- 所有字段都要有默认值
- 如果有其它参数，可以再加，基本分页对象就只包含所有分页都需要用到的字段

### 分页返回对象
```javascript
{
    // 当前页数
    "currentPage": 1,
    // 是否有下一页
    "hasNext": true,
    // 是否有前一页
    "hasPrev": true,
    // 总的对象数量
    "totalNum": 30,
    // 总的页数
    "totalPage": 2,
    // 对象列表
    "items": [
        {},
        {},
        {}
    ]
}
```


## 状态码（Status Codes）

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）

- 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）
- 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功
- 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
- 204 NO CONTENT - [DELETE]：用户删除数据成功
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的
- 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）
- 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的
- 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的
- 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）
- 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的
- 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误
- 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功


## 错误处理（Error handling）

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可
```javascript
{
    error: "Invalid API key"
}
```


## 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范

- GET`/collection`：返回资源对象的列表（数组）
- GET`/collection/resource`：返回单个资源对象
- POST`/collection`：返回新生成的资源对象
- PUT`/collection/resource`：返回完整的资源对象
- PATCH`/collection/resource`：返回完整的资源对象
- DELETE`/collection/resource`：返回一个空文档

注意：**返回值永远是JSON对象**，严禁返回如下内容（在**很多公司都会这样干，这是严重不对的**）
```javascript
{
    "msg": "test",
    "code": 20,
    "data": { // 实际的数据
        "aaa": 1,
        "bbb": 2
    }
}
```

## JSON格式规范

返回的JSON格式中的变量名有很多种方式，采用首字母小写的驼峰命名法

### 反例
```javascript
{
    "test_field": "testFiled"
}
```

### 正例
```javascript
{
    "testField": "testFiled"
}
```


## Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档
```javascript
{
    "link": {
        "rel":   "collection https://www.example.com/zoos",
        "href":  "https://api.example.com/zoos",
        "title": "List of zoos",
        "type":  "application/vnd.yourformat+json"
    }
}
```
上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问api.github.com会得到一个所有可用API的网址列表
```javascript
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
```
从上面可以看到，如果想获取当前用户的信息，应该去访问api.github.com/user，然后就得到了下面结果
```javascript
{
  "message": "Requires authentication",
  "documentation_url": "https://developer.github.com/v3"
}
```
上面代码表示，服务器给出了提示信息，以及文档的网址


## 参考资料

- [REST接口设计规范总结](https://juejin.im/entry/5b0aa44b6fb9a07acf569ce8)
- [RESTful API规范](https://i6448038.github.io/2017/06/28/rest-%E6%8E%A5%E5%8F%A3%E8%A7%84%E8%8C%83/)
- [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
