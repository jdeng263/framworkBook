# API规范最佳实践

## API设计的基本要求
> 1.当标准合理的时候遵守标准。\
> 2.API应该对程序员友好，并且在浏览器地址栏容易输入。\
> 3.API应该简单，直观，容易使用的同时优雅。\
> 4.API应该具有足够的灵活性来支持上层ui。\
> 5.API设计权衡上述几个原则。

## 协议HTTPS

> 增加网站的安全是非常重要的。特别如果你提供的是公开 API，用户的信息泄露或者被攻击会严重影响网站

## API 地址和版本
参考例子
> 1.https://api.hztianque.com/v3\
> 2.https://hztianque.com/api/v3\
> 3.或是把版本号添加到head


## API 路径规则

路径又称"终点"（endpoint），表示API的具体网址。
在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。
举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。

> https://api.example.com/v1/products\
https://api.example.com/v1/users\
https://api.example.com/v1/employees

## HTTP请求方式

对于资源的具体操作类型，由HTTP动词表示。
常用的HTTP动词有下面四个（括号里是对应的SQL命令）。
- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
- DELETE（DELETE）：从服务器删除资源。

### 下面是一些例子。

- GET /product：列出所有商品
- POST /product：新建一个商品
- GET /product/ID：获取某个指定商品的信息
- PUT /product/ID：更新某个指定商品的信息
- DELETE /product/ID：删除某个商品
- GET /product/ID/purchase ：列出某个指定商品的所有投资者
- get /product/ID/purchase/ID：获取某个指定商品的指定投资者信息

## 过滤信息

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。
- ?limit=10：指定返回记录的数量
- ?offset=10：指定返回记录的开始位置。
- ?page=2&per_page=100：指定第几页，以及每页的记录数。
- ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
- ?producy_type=1：指定筛选条件

## API 传入参数

参入参数分为4种类型：

> 地址栏参数
> * restful 地址栏参数 /api/v1/product/122 122为产品编号，获取产品为122的信息
> * get方式的查询字串 见过滤信息小节\
> 请求body数据\
> cookie\
> request header

cookie和header 一般都是用于OAuth认证的2种途径

## schema
对于响应返回的格式，JSON 因为它的可读性、紧凑性以及多种语言支持等优点，成为了 HTTP API 最常用的返回格式。因此，最好采用 JSON 作为返回内容的格式。如果用户需要其他格式，比如 xml，应该在请求头部 Accept 中指定。对于不支持的格式，服务端需要返回正确的 status code，并给出详细的说明。
```json
{
    code:0 //返回码0表示成功
    message:"success" //开发使用描述信息
    data{  // 请求数据，对象或数组均可
        user_id: 123,
        user_name: "tutuge",
        user_avatar_url: "http://tutuge.me/avatar.jpg"
    }
    Errors[
      {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot have fancy characters"
      }
    ]
}



```

## 以资源为中心的URL设计

## 分页 Pagination

```h
curl 'https://api.hztianque.com/user/repos?page=2&per_page=100'
```
在 APP 后端分页设计 中提到过，分页布局一般分为两种，一种是在 Web 端比较常见的有底部分页栏的电梯式分页，另一种是在 APP 中比较常见的上拉加载更多的流式分页。这两种分页的 API 到底该如何设计呢？

电梯式分页需要提供page（页数）和pre_page（每页的数量）。例如：


> /users/?page=2&pre_page=20

而服务端则需要额外返回total_count（总记录数），以及可选的当前页数、每页的数量（这两个与客户端提交的相同）、总页数、是否有下一页、是否有上一页（这三个都可以通过总记录数计算出）。例如：

```json
{
    "pagination": {
       "previous": 1,
       "next": 3,
       "current": 2,
       "per_page": 20,
       "total": 200,
       "pages": 10
    },
    "data": {}
}
```
流式布局也完全可以使用这种方式，并且不需要查询总记录数（好处是减少一次数据库操作，坏处时客户端需要多请求一次才能判断是否到最后一页）。但是会出现数据重复和缺失的情况，所以更推荐使用游标分页。

游标分页需要提供cursor(下一页的起点游标) 和limit(数量) 参数。例如：


> /articles/?cursor=2015-01-01 15:20:30&limit=10

如果文章列表默认是以创建时间为倒序排列的，那么cursor就是当前列表最后一条的创建时间（第一页为当前时间）。

服务端需要返回的数据也很简单，只需要以此游标为起点的总记录数和下一个起点游标就可以了。例如：





```json

{
    "pagination": {
       "next": "2015-01-01 12:20:30",
       "limit": 10,
       "total": 100,
    },
    "data": {}
}
```

如果total小于limit，就说明已经没有数据了。

流式布局的分页 API 还有一种情况很常见，就是下拉刷新的增量更新。它的业务逻辑正好和游标分页相反，但是参数基本一样：

> /articles/?cursor=2015-01-01 15:20:30&limit=20

返回数据有两种可能，一种是增量更新的数据小于指定的数量，就直接将全部数据返回（这个数量可以设置的相对大一些），客户端会将这些增量更新的数据添加在已有列表的顶部。但是如果增量更新的数据要大于指定的数量，就会只返回最新的 n 条数据作为第一页，这时候客户端需要清空之前的列表。例如：

```json
{
    "pagination": {
       "limit": 20,
       "total": 100,
    },
    "data": {}
}
```

如果total大于limit，说明增量的数据太多所以只返回了第一页，需要清空旧的列表。


## 选择合适的状态码
```h
200 ok ?- 成功返回状态，对应，GET,PUT,PATCH,DELETE.
201 created??-?成功创建。
304 not modified ??-?HTTP缓存有效。
400 bad request ??-?请求格式错误。
401 unauthorized ??-?未授权。
403 forbidden ??-?鉴权成功，但是该用户没有权限。
404 not found - 请求的资源不存在
405 method not allowed - 该http方法不被允许。
410 gone - 这个url对应的资源现在不可用。
415 unsupported media type - 请求类型错误。
422 unprocessable entity - 校验错误时用。
429 too many request - 请求过多。
```


## 错误处理：给出详细的信息

单一的错误
```json
{
  "code" : 1234,
  "message" : "Something bad happened :-(",
  "description" : "More details about the error here"
}
```

提交验证

```json
{
  "code" : 1024,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot have fancy characters"
    },
    {
       "code" : 5622,
       "field" : "password",
       "message" : "Password cannot be blank"
    }
  ]
}
```



## 验证和授权
Token 和 Sign

API 需要设计成无状态，所以客户端在每次请求时都需要提供有效的 Token 和 Sign，在我看来它们的用途分别是：

- Token 用于证明请求所属的用户，一般都是服务端在登录后随机生成一段字符串（UUID）和登录用户进行绑定，再将其返回给客户端。Token 的状态保持一般有两种方式实现：一种是在用户每次操作都会延长或重置 TOKEN 的生存时间（类似于缓存的机制），另一种是 Token 的生存时间固定不变，但是同时返回一个刷新用的 Token，当 Token 过期时可以将其刷新而不是重新登录。
- Sign 用于证明该次请求合理，所以一般客户端会把请求参数拼接后并加密作为 Sign 传给服务端，这样即使被抓包了，对方只修改参数而无法生成对应的 Sign 也会被服务端识破。当然也可以将时间戳、请求地址和 Token 也混入 Sign，这样 Sign 也拥有了所属人、时效性和目的地。

## 限流 rate limit
```javascript
X-RateLimit-Limit: 用户每个小时允许发送请求的最大值
X-RateLimit-Remaining：当前时间窗口剩下的可用请求数目
X-RateLimit-Rest: 时间窗口重置的时候，到这个时间点可用的请求数量就会变成 X-RateLimit-Limit 的值
```

## 编写优秀的文档
- 让你的文档对那些未经认证的开发者也可用
- 不要使用文档自动化生成器，即便你用了，你也要保证自己审阅过并让它具有更好的版式。
- 不要截断示例中请求与响应的内容，要展示完整的东西。并在文档中使用高亮语法。
- 文档化每一个端点所预期的响应代码和可能的错误消息，和在什么情况下会产生这些的错误消息
