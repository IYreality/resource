# 当前版本

默认情况下，所有到https://api.myblog.com的请求都会收到REST API的v3版本。我们建议您通过Accept 显式地请求这个版本。

```
Accept: application/vnd.myblog.v3+json
```

## 版本历史

| 日期       | 版本号 | 作者  | 备注       |
| ---------- | ------ | ----- | ---------- |
| 2019.11.17 | 1.0    | Ailsa | 新版本发布 |
| 2019.11.18 | 1.1    | Ailsa | 新版本发布 |

# 文档介绍

本文档的接口遵循RESTful设计风格。

## 登录认证流程

基于JWT认证机制，实现登录认证流程。

# 模式

所有的API访问都是通过HTTPS进行的，并通过https://api.myblog.com进行访问。所有数据都以JSON的形式发送和接收。

```json
curl -i https://api.myblog.com/users/octocat/orgs
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 12 Oct 2012 23:33:14 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Status: 200 OK
ETag: "a00049ba79152d03380c34652f2cb612"
X-myblog-Media-Type: myblog.v3
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4987
X-RateLimit-Reset: 1350085394
Content-Length: 5
Cache-Control: max-age=0, private, must-revalidate
X-Content-Type-Options: nosniff
```

空白字段被包含为null，而不是被忽略。

所有时间戳返回ISO 8601格式:

```
YYYY-MM-DDTHH: MM: SSZ
```

接口返回一共有三种情况：

1. 操作成功后返回，范例：

```json
{
  "code": "200",
  "msg": "SUCCESS"
} 
```

1. 成功返回数据，范例：

```json
{
  "data": {
    "name": "minhow",
    "age": "18"
  },
  "code": "200",
  "msg": "SUCCESS"
}
```

1. 错误返回，范例：

```json
{
  "err_code": "1001",
  "err_msg": "wrong,again！"
}
```

## 总结陈述

当您获取资源列表时，响应包括该资源的属性子集。 这是资源的“摘要”表示。 （API提供的某些属性在计算上很昂贵。出于性能原因，摘要表示排除了那些属性。要获取这些属性，请获取“详细”表示。）

**示例**：当您获得存储库列表时，您将获得每个存储库的摘要表示。 在这里，我们获取octokit组织拥有的存储库列表：

```
GET / orgs / octokit / repos
```

## 详细表示

当您获取单个资源时，响应通常包括该资源的所有属性。 这是资源的“详细”表示。 （请注意，授权有时会影响表示中包含的详细信息的数量。）

**示例**：获得单个存储库时，将获得存储库的详细表示。 在这里，我们获取octokit / octokit.rb存储库：

```
GET /repos/octokit/octokit.rb
```

# 身份验证

有两种方法可以通过myblog API v3进行身份验证。 在某些地方，需要身份验证的请求将返回<kbd>404 Not Found</kbd>，而不是<kbd>403 Forbidden </kbd>。 这是为了防止私有存储库意外泄露给未经授权的用户。

## 基本认证

```
curl -u "username" https://api.myblog.com
```

## Token 验证

`Token`访问有效期为两个小时，刷新有效期为两周。

## 刷新Token值

## 请求说明

请求方式：PATCH
返回结果：

```json
{
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vdm95YWdlLmRldi9hdXRob3JpemF0aW9ucy9yZWZyZXNoLXRva2VucyIsImlhdCI6MTQ4OTk4Nzg0OCwiZXhwIjoxNDg5OTg3OTc3LCJuYmYiOjE0ODk5ODc5MTcsImp0aSI6IlRvNmxzamhwTTNpcmhRQlAiLCJ1dWlkIjoiNWZlYzI0NzAifQ.hgZsQq5rT5VXAwUilEv5P1JIhLrctJPKAkKWBSqwu3c"
  },
  "code": "200",
  "msg": "SUCCESS"
}
```

### 返回参数

| 字段  | 字段类型 | 字段说明 |
| ----- | -------- | -------- |
| token | string   | token值  |

# 登录

## 请求说明

请求方式：POST

请求URL:    login

请求参数：

| 字段     | 字段类型 | 字段说明 |
| -------- | -------- | -------- |
| email    | string   | 邮箱     |
| username | string   | 账户     |
| password | string   | 密码     |

返回结果：

```json
{
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vc2FsZS1hcGkuZGV2L2xvZ2luIiwiaWF0IjoxNDkxNTMyOTI4LCJleHAiOjE0OTIyNTI5MjgsIm5iZiI6MTQ5MTUzMjkyOCwianRpIjoiN1hCUXdwN1FHZmxUdHVVQiIsInV1aWQiOiI1MDZjYWY3MCJ9.FyyXagHtBfDBtMJZPV_hm2q6CVULpY63JPDGDHXc"
  },
  "code": "200",
  "msg": "SUCCESS"
}
```

## 登录限制失败

使用无效凭据进行身份验证将返回<kbd>401</kbd>>未经授权：

```
curl -i https://api.myblog.com -u foo:bar
HTTP/1.1 401 Unauthorized
{
  "message": "Bad credentials",
  "documentation_url": "https://developer.myblog.com/v3"
}
```

在短时间内检测到多个具有无效凭据的请求后，API会暂时拒绝该用户的所有身份验证尝试（包括具有有效凭据的请求），并设置<kbd>403</kbd>禁止：

```
curl -i https://api.myblog.com -u valid_username:valid_password
HTTP/1.1 403 Forbidden
{
  "message": "Maximum number of login attempts exceeded. Please try again later.",
  "documentation_url": "https://developer.myblog.com/v3"
}
```

# 参数

许多API方法采用可选参数。对于<kbd>GET</kbd>请求，任何未在路径中指定为段的参数都可以作为HTTP查询字符串参数传递:

```
curl -i "https://api.myblog.com/repos/vmg/redcarpet/issues?state=closed"
```

在本例中，为路径中的:owner和:repo参数提供了'vmg'和'redcarpet'值，而:state则在查询字符串中传递。

对于POST、PATCH、PUT和DELETE请求，URL中没有包含的参数应该用“application/ JSON”的内容类型编码为JSON:

```
curl -i -u username -d '{"scopes":["public_repo"]}' https://api.myblog.com/authorizations
```

# 客户端错误

在接收请求体的API调用上有三种可能的客户端错误类型:

1. 发送无效的JSON将导致<kbd>400 Bad Request</kbd>>响应。```

   ```json
   HTTP/1.1 400 Bad Request
   Content-Length: 35
   
   {"message":"Problems parsing JSON"}
   ```

2. 发送错误类型的JSON值将导致<kbd>400 Bad Request</kbd>响应。```

   ```json
   HTTP/1.1 400 Bad Request
   Content-Length: 40
   
   {"message":"Body should be a JSON object"}
   ```

3. Sending invalid fields will result in a<kbd>422 Unprocessable Entity</kbd> response.```

   ```json
   HTTP/1.1 422 Unprocessable Entity
   Content-Length: 149
   
   {
     "message": "Validation Failed",
     "errors": [
       {
         "resource": "Issue",
         "field": "title",
         "code": "missing_field"
       }
     ]
   }
   ```

所有的错误对象都有资源和字段属性，这样您的客户端就可以知道问题是什么。还有一个错误代码让您知道字段出了什么问题。以下是可能的验证错误代码:

| 错误           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| missing        | 意味着资源不存在。                                           |
| missing_field  | 意味着资源上的必填字段没有设置。                             |
| invalid        | 这意味着字段的格式是无效的。该资源的文档应该能够提供更具体的信息。 |
| already_exists | 这意味着另一个资源具有与该字段相同的值。这可能发生在必须具有某些唯一键的资源中(例如标签名称)。 |

# HTTP重定向

API v3在适当的地方使用HTTP重定向。客户端应该假设任何请求都可能导致重定向。接收HTTP重定向不是一个错误，客户端应该遵循这个重定向。重定向响应将有一个Location头字段，其中包含客户端应向其重复请求的资源的URI。

| 状态码  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| 301     | 永久重定向。用于发出请求的URI已被Location头字段中指定的URI取代。这个请求和以后对这个资源的所有请求都应该指向新的URI。 |
| 302,307 | 临时重定向。请求应该对Location头字段中指定的URI进行逐字重复，但是客户端应该继续使用原始的URI处理未来的请求。 |

其他重定向状态码可以根据HTTP 1.1规范使用。

# HTTP动作

在可能的情况下，API v3努力为每个操作使用适当的HTTP动词。

| 动作   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| HEAD   | 可以对任何资源发出，以获取HTTP头信息。                       |
| GET    | 用于检索资源。                                               |
| POST   | 用于创建资源。                                               |
| PATCH  | 用于用部分JSON数据更新资源的补丁。例如，问题资源具有标题和正文属性。补丁请求可以接受一个或多个属性来更新资源。PATCH是一个相对较新的、不常见的HTTP动词，因此资源端点也接受POST请求。 |
| PUT    | 用于替换资源或集合。对于没有body属性的PUT请求，请确保将Content-Length头设置为零。 |
| DELETE | 用于删除资源。                                               |

# 超媒体

所有资源可以有一个或多个*_url属性链接到其他资源。这意味着提供显式的url，这样适当的API客户机就不需要自己构造url。强烈建议API客户端使用这些。这样做将使开发人员更容易升级API。所有url都应该是正确的RFC 6570 URI模板。

然后你可以使用uri_template gem扩展这些模板:

```json
>> tmpl = URITemplate.new('/notifications{?since,all,participating}')
>> tmpl.expand
=> "/notifications"

>> tmpl.expand :all => 1
=> "/notifications?all=1"

>> tmpl.expand :all => 1, :participating => 1
=> "/notifications?all=1&participating=1"
```

# 分页

默认情况下，返回多个条目的请求将被分页为30个条目。可以使用?page参数指定其他页面。对于某些资源，还可以使用?per_page参数将自定义页面大小设置为100。请注意，出于技术原因，并非所有端点都尊重?per_page参数，请参见==“事件”==。

```
curl 'https://api.myblog.com/user/repos?page=2&per_page=100'
```

请注意，页面编号是基于1的，删除?page参数将返回第一个页面。

有关分页的更多信息，请参阅我们的分页遍历指南。

## 链接头部：

> 注意:使用链接头值而不是构造您自己的url来形成调用是很重要的。

链接头包含分页信息:

```
Link: <https://api.myblog.com/user/repos?page=3&per_page=100>; rel="next",
  <https://api.myblog.com/user/repos?page=50&per_page=100>; rel="last"
```

该示例包含一个换行符，以提高可读性。

这个链接响应头包含一个或多个超媒体链接关系，其中一些可能需要作为URI模板展开。

可能的rel值为:

| 名字  | 描述                           |
| ----- | ------------------------------ |
| next  | 直接下一页结果的链接关系。     |
| last  | 最后一页结果的链接关系。       |
| first | 首先，链接关系的第一页的结果。 |
| prev  | 前一页搜索结果的链接关系。     |

# 速度限制

对于使用基本身份验证或OAuth的API请求，每小时最多可以发出5000个请求。无论使用的是基本身份验证还是OAuth令牌，经过身份验证的请求都与经过身份验证的用户相关联。这意味着，当用户授权的所有OAuth应用程序使用同一用户拥有的不同令牌进行身份验证时，它们共享相同的配额，即每小时5000个请求。

对于未经身份验证的请求，速率限制允许每小时最多60个请求。未经身份验证的请求与原始IP地址关联，而与发出请求的用户无关。

注意，搜索API有自定义的速率限制规则。

任何API请求返回的HTTP头信息都会显示你当前的速率限制状态:

```json
curl -i https://api.myblog.com/users/octocat
HTTP/1.1 200 OK
Date: Mon, 01 Jul 2013 17:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 56
X-RateLimit-Reset: 1372700873
```

| 头部名称              | 描述                                          |
| --------------------- | --------------------------------------------- |
| X-RateLimit-Limit     | 每小时允许发出的最大请求数。                  |
| X-RateLimit-Remaining | 保留当前速率限制窗口中保留的请求数。          |
| X-RateLimit-Reset     | 重置当前速率限制窗口在UTC历元秒内重置的时间。 |

如果你需要不同格式的时间，任何现代编程语言都可以完成这项工作。例如，如果您打开web浏览器上的控制台，您可以很容易地将重置时间作为JavaScript Date对象获得。

```
new Date(1372700873 * 1000)
// => Mon Jul 01 2013 13:47:53 GMT-0400 (EDT)
```

如果超过速率限制，则返回错误响应:

```json
HTTP/1.1 403 Forbidden
Date: Tue, 20 Aug 2013 14:50:41 GMT
Status: 403 Forbidden
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1377013266
{
   "message": "API rate limit exceeded for xxx.xxx.xxx.xxx. (But here's the good news: Authenticated requests get a higher rate limit. Check out the documentation for more details.)",
   "documentation_url": "https://developer.myblog.com/v3/#rate-limiting"
}
```

您可以检查您的速率限制状态，而不会导致API命中。

## 速率限制

如果您的OAuth应用程序需要使用更高的速率限制进行未经身份验证的调用，您可以将应用程序的客户端ID和secret作为查询字符串的一部分传递。

```json
curl -i 'https://api.myblog.com/users/whatever?client_id=xxxx&client_secret=yyyy'
HTTP/1.1 200 OK
Date: Mon, 01 Jul 2013 17:27:06 GMT
Status: 200 OK
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4966
X-RateLimit-Reset: 1372700873
```

> 注意:永远不要与任何人共享您的客户端机密，也不要将它包含在客户端浏览器代码中。仅对服务器到服务器的调用使用这里显示的方法。

如果使用基本身份验证或OAuth超过了速率限制，则可以通过缓存API响应和使用条件请求来修复此问题。

## 滥用率限制

为了在myblog上提供高质量的服务，在使用API时，可能会对某些操作施加额外的速率限制。例如，使用API快速创建内容、主动轮询而不是使用webhook、发出多个并发请求或重复请求计算开销大的数据可能会导致滥用率限制。

滥用率限制并不是为了干扰API的合法使用。你的正常速度限制应该是你唯一的目标限制。为了确保你是一个良好的API公民，请查看我们的最佳实践指南。

如果你的应用程序触发了这个速率限制，你将收到一个有用的响应:

```json
HTTP/1.1 403 Forbidden
Content-Type: application/json; charset=utf-8
Connection: close
{
  "message": "You have triggered an abuse detection mechanism and have been temporarily blocked from content creation. Please retry your request again later.",
  "documentation_url": "https://developer.myblog.com/v3/#abuse-rate-limits"
}
```



# 用户代理要求

所有API请求必须包含一个有效的用户代理头。没有用户代理头的请求将被拒绝。我们要求您使用您的myblog用户名，或您的应用程序的名称，作为用户-代理头值。这允许我们在有问题时联系您。

这里有一个例子:

```
User-Agent: Awesome-Octocat-App
```

默认情况下，cURL发送一个有效的用户-代理头文件。如果你通过cURL(或另一个客户端)提供了一个无效的用户代理报头，你将收到一个403禁止响应:

```json
curl -iH 'User-Agent: ' https://api.myblog.com/meta
HTTP/1.0 403 Forbidden
Connection: close
Content-Type: text/html
Request forbidden by administrative rules.
Please make sure your request has a User-Agent header.
Check https://developer.myblog.com for other possible causes.
```

# 有条件的请求

大多数响应返回一个ETag头。许多响应还返回最后修改的标题。您可以使用这些标头的值，分别使用If-None-Match和If-Modified-Since标头向这些资源发出后续请求。如果资源没有更改，服务器将返回一个304 not Modified。

> 注意:发出有条件的请求并收到304个响应并不会影响您的速率限制，因此我们鼓励您尽可能使用它。

```json
curl -i https://api.myblog.com/user
HTTP/1.1 200 OK
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 200 OK
Vary: Accept, Authorization, Cookie
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4996
X-RateLimit-Reset: 1372700873
curl -i https://api.myblog.com/user -H 'If-None-Match: "644b5b0155e6404a9cc4bd9d8b1ae730"'
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
ETag: "644b5b0155e6404a9cc4bd9d8b1ae730"
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 304 Not Modified
Vary: Accept, Authorization, Cookie
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4996
X-RateLimit-Reset: 1372700873
curl -i https://api.myblog.com/user -H "If-Modified-Since: Thu, 05 Jul 2012 15:31:30 GMT"
HTTP/1.1 304 Not Modified
Cache-Control: private, max-age=60
Last-Modified: Thu, 05 Jul 2012 15:31:30 GMT
Status: 304 Not Modified
Vary: Accept, Authorization, Cookie
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4996
X-RateLimit-Reset: 1372700873
```



# 跨源资源共享

对于来自任何来源的AJAX请求，该API支持跨来源资源共享(CORS)。您可以阅读CORS W3C推荐标准，或者HTML 5安全指南中的介绍。

下面是一个来自浏览器的请求示例，地址是http://example.com:

```json
curl -i https://api.myblog.com -H "Origin: http://example.com"
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Access-Control-Expose-Headers: ETag, Link, X-myblog-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
```

CORS请求:

```
curl -i https://api.myblog.com -H "Origin: http://example.com" -X OPTIONS
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-myblog-OTP, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-myblog-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval
Access-Control-Max-Age: 86400
```

# JSON-P回调

可以向任何GET调用发送?回调参数，以便将结果包装在JSON函数中。这通常用于浏览器希望通过绕过跨域问题在web页面中嵌入myblog内容时。响应包括与常规API相同的数据输出，以及相关的HTTP头信息。

```json
curl https://api.myblog.com?callback=foo
/**/foo({
  "meta": {
    "status": 200,
    "X-RateLimit-Limit": "5000",
    "X-RateLimit-Remaining": "4966",
    "X-RateLimit-Reset": "1372700873",
    "Link": [ // pagination headers and other links
      ["https://api.myblog.com?page=2", {"rel": "next"}]
    ]
  },
  "data": {
    // the data
  }
})
```

您可以编写一个JavaScript处理程序来处理回调。这里有一个最小的例子，你可以尝试:

```javascript
<html>
<head>
<script type="text/javascript">
function foo(response) {
  var meta = response.meta;
  var data = response.data;
  console.log(meta);
  console.log(data);
}

var script = document.createElement('script');
script.src = 'https://api.myblog.com?callback=foo';

document.getElementsByTagName('head')[0].appendChild(script);
</script>
</head>

<body>
  <p>Open up your browser's console.</p>
</body>
</html>
```

所有报头都是与HTTP报头相同的字符串值，但有一个明显的例外:Link。链接头是预先为您解析的，并通过一个[url，选项]元组数组。

链接是这样的:

```
Link: <url1>; rel="next", <url2>; rel="foo"; bar="baz"
```

...回调输出如下:

```json
{
  "Link": [
    [
      "url1",
      {
        "rel": "next"
      }
    ],
    [
      "url2",
      {
        "rel": "foo",
        "bar": "baz"
      }
    ]
  ]
}
```

# 时区

有些创建新数据的请求(如创建新的提交)允许您在指定或生成时间戳时提供时区信息。我们按照优先级的顺序应用以下规则来确定API调用的时区信息。

- 显式地提供带有时区信息的ISO 8601时间戳
- 使用时区报头
- 为用户使用最后一个已知时区
- 默认为UTC，没有其他时区信息

## 显式地提供带有时区信息时间戳

对于允许指定时间戳的API调用，我们使用精确的时间戳。提交API就是一个例子。

这些时间戳看起来有点像<kbd>2014-02-27T15:05:06+01:00</kbd>。还可以查看这个示例，了解如何指定这些时间戳。

## 使用时区报头

可以提供一个时区报头，它根据Olson数据库中的名称列表定义一个时区。

```json
curl -H "Time-Zone: Europe/Amsterdam" -X POST https://api.myblog.com/repos/myblog/linguist/contents/new_file.md
```

这意味着，当您的API调用在这个头定义的时区中进行时，我们将生成一个时间戳。例如，Contents API为每个添加或更改生成一个git提交，并使用当前时间作为时间戳。此标头将确定用于生成当前时间戳的时区。

## 为用户使用最后一个已知时区

如果没有指定时区报头，并且您对API进行了身份验证调用，那么我们将为经过身份验证的用户使用最后一个已知的时区。当你浏览myblog网站时，最新的时区就会更新。

## 默认为UTC，没有其他时区信息

如果上面的步骤没有得到任何信息，我们将使用UTC作为时区来创建git提交。



# 附录

## 请求返回值表格

**正常**

| 返回 | 说明                                               |
| ---- | -------------------------------------------------- |
| 200  | 请求成功。                                         |
| 202  | 任务提交成功，当前系统繁忙，下发的任务会延迟处理。 |
| 204  | 任务提交成功。                                     |

**异常**

| 返回值                            | 说明                                                     |
| --------------------------------- | -------------------------------------------------------- |
| 300 multiple choices              | 被请求的资源存在多个可供选择的响应。                     |
| 400 Bad Request                   | 服务器未能处理请求。                                     |
| 401 Unauthorized                  | 被请求的页面需要用户名和密码。                           |
| 403 Forbidden                     | 对被请求页面的访问被禁止。                               |
| 404 Not Found                     | 服务器无法找到被请求的页面。                             |
| 405 Method Not Allowed            | 请求中指定的方法不被允许。                               |
| 406 Not Acceptable                | 服务器生成的响应无法被客户端所接受。                     |
| 407 Proxy Authentication Required | 用户必须首先使用代理服务器进行验证，这样请求才会被处理。 |
| 408 Request Timeout               | 请求超出了服务器的等待时间。                             |
| 409 Conflict                      | 由于冲突，请求无法被完成。                               |
| 500 Internal Server Error         | 请求未完成。服务异常。                                   |
| 501 Not Implemented               | 请求未完成。服务器不支持所请求的功能。                   |
| 502 Bad Gateway                   | 请求未完成。服务器从上游服务器收到一个无效的响应。       |
| 503 Service Unavailable           | 请求未完成。系统暂时异常。                               |
| 504 Gateway Timeout               | 网关超时。                                               |

**状态码表格**

| 状态码 | 编码                            | 状态说明                                                     |
| ------ | ------------------------------- | ------------------------------------------------------------ |
| 100    | Continue                        | 继续请求。这个临时响应用来通知客户端，它的部分请求已经被服务器接收，且仍未被拒绝。 |
| 101    | Switching Protocols             | 切换协议。只能切换到更高级的协议。例如，切换到HTTP的新版本协议。 |
| 201    | Created                         | 创建类的请求完全成功。                                       |
| 202    | Accepted                        | 已经接受请求，但未处理完成。                                 |
| 203    | Non-Authoritative Information   | 非授权信息，请求成功。                                       |
| 204    | NoContent                       | 请求完全成功，同时HTTP响应不包含响应体。在响应OPTIONS方法的HTTP请求时返回此状态码。 |
| 205    | Reset Content                   | 重置内容，服务器处理成功。                                   |
| 206    | Partial Content                 | 服务器成功处理了部分GET请求。                                |
| 300    | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择。 |
| 301    | Moved Permanently               | 永久移动，请求的资源已被永久的移动到新的URI，返回信息会包括新的URI。 |
| 302    | Found                           | 资源被临时移动。                                             |
| 303    | See Other                       | 查看其它地址。使用GET和POST请求查看。                        |
| 304    | Not Modified                    | 所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。 |
| 305    | Use Proxy                       | 所请求的资源必须通过代理访问。                               |
| 306    | Unused                          | 已经被废弃的HTTP状态码。                                     |
| 400    | BadRequest                      | 非法请求。建议直接修改该请求，不要重试该请求。               |
| 401    | Unauthorized                    | 在客户端提供认证信息后，返回该状态码，表明服务端指出客户端所提供的认证信息不正确或非法。 |
| 402    | Payment Required                | 保留请求。                                                   |
| 403    | Forbidden                       | 请求被拒绝访问。返回该状态码，表明请求能够到达服务端，且服务端能够理解用户请求，但是拒绝做更多的事情，因为该请求被设置为拒绝访问，建议直接修改该请求，不要重试该请求。 |
| 404    | NotFound                        | 所请求的资源不存在。建议直接修改该请求，不要重试该请求。     |
| 405    | MethodNotAllowed                | 请求中带有该资源不支持的方法。建议直接修改该请求，不要重试该请求。 |
| 406    | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求。                 |
| 407    | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权。 |
| 408    | Request Time-out                | 服务器等候请求时发生超时。客户端可以随时再次提交该请求而无需进行任何更改。 |
| 409    | Conflict                        | 服务器在完成请求时发生冲突。返回该状态码，表明客户端尝试创建的资源已经存在，或者由于冲突请求的更新操作不能被完成。 |
| 410    | Gone                            | 客户端请求的资源已经不存在。返回该状态码，表明请求的资源已被永久删除。 |
| 411    | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息。     |
| 412    | Precondition Failed             | 未满足前提条件，服务器未满足请求者在请求中设置的其中一个前提条件。 |
| 413    | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息。 |
| 414    | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理。             |
| 415    | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式。                           |
| 416    | Requested range not satisfiable | 客户端请求的范围无效。                                       |
| 417    | Expectation Failed              | 服务器无法满足Expect的请求头信息。                           |
| 422    | UnprocessableEntity             | 请求格式正确，但是由于含有语义错误，无法响应。               |
| 429    | TooManyRequests                 | 表明请求超出了客户端访问频率的限制或者服务端接收到多于它能处理的请求。建议客户端读取相应的Retry-After首部，然后等待该首部指出的时间后再重试。 |
| 500    | InternalServerError             | 表明服务端能被请求访问到，但是不能理解用户的请求。           |
| 501    | Not Implemented                 | 服务器不支持请求的功能，无法完成请求。                       |
| 502    | Bad Gateway                     | 充当网关或代理的服务器，从远端服务器接收到了一个无效的请求。 |
| 503    | ServiceUnavailable              | 被请求的服务无效。建议直接修改该请求，不要重试该请求。       |
| 504    | ServerTimeout                   | 请求在给定的时间内无法完成。客户端仅在为请求指定超时（Timeout）参数时会得到该响应。 |
| 505    | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理。             |

## 任务类接口

**正常响应要素说明**

| 名称   | 参数类型 | 说明                                                         |
| ------ | -------- | ------------------------------------------------------------ |
| job_id | String   | 提交任务成功后返回的任务ID，用户可以使用该ID对任务执行情况进行查询。如何根据job_id来查询Job的执行状态，请参考查询Job状态。 |

**异常响应要素说明**

| 名称  | 参数类型     | 说明                                                         |
| ----- | ------------ | ------------------------------------------------------------ |
| error | 字典数据结构 | 提交任务异常是返回的异常信息，详情请参见 error 数据结构说明。 |

**error 数据结构说明**

| 名称    | 参数类型 | 说明                   |
| ------- | -------- | ---------------------- |
| message | String   | 任务异常错误信息描述。 |
| code    | String   | 任务异常错误信息编码。 |

## 响应样例

**正常响应：**

```json
{ 
    "job_id": "70a599e0-31e7-49b7-b260-868f441e862b", 
} 
```

**异常响应：**

```json
{ 
    "error": {"message": "", "code": XXX}
} 
```

## 批量接口

**正常响应要素说明**

| 名称     | 参数类型     | 说明                                                         |
| -------- | ------------ | ------------------------------------------------------------ |
| response | 列表数据结构 | 提交请求成功后返回的响应列表，详情请参见下面response数据结构说明。 |

**response数据结构说明**

| 名称 | 参数类型 | 说明               |
| :--- | -------- | ------------------ |
| id   | String   | 操作成功的虚拟机id |

**异常响应要素说明**

| 名称          | 参数类型      | 说明                                                         |
| ------------- | ------------- | ------------------------------------------------------------ |
| error         | 字典-数据结构 | 批量请求异常时返回的整体异常信息，详情请参见 error 数据结构。 |
| internalError | 列表-数据结构 | 批量请求处理中，每一个单个请求的具体异常信息，详情请参见 internalError 数据结构说明。 |

**error 数据结构说明**

| 名称    | 参数类型 | 说明                           |
| ------- | -------- | ------------------------------ |
| message | String   | 批量操作整体异常错误信息描述。 |
| code    | String   | 批量操作整体异常错误信息编码。 |

**internalError数据结构说明**

| 名称          | 参数类型 | 说明                                 |
| ------------- | -------- | ------------------------------------ |
| id            | String   | 具体单个请求操作失败的虚拟机id       |
| error_message | String   | 具体单个请求操作失败的错误信息描述。 |
| error_code    | String   | 具体单个请求操作失败的错误信息编码。 |

## 响应样例

**正常响应：**

```json
{ 
    "response": [
                  {
                    "id": "616fb98f-46ca-475e-917e-2563e5a8cd19"   
                  },
                  {
                    "id": "516fb98f-46ca-475e-917e-2563e5a8cd12"   
                  }
               ]
}
```

**异常响应：**

```json
{
     "error": {
                 "code": "Ecs.xxxx",
                 "message": "xxxxxxxxxxxxxxx" 
               },
     "internalError": [
                 {
                    "id": "616fb98f-46ca-475e-917e-2563e5a8cd19",
                    "error_code": "ECS.XXXX",
                    "error_message": "xxxxxxxxxxxxxxx" 
                  },
                 {
                     "id": "516fb98f-46ca-475e-917e-2563e5a8cd12",
                     "error_code": "ECS.XXXX",
                     "error_message": "xxxxxxxxxxxxxxx" 
                 }
              ]
}
```
