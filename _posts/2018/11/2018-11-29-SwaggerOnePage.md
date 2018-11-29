---
layout: post
title: Swagger In One Page
subtitle: Generate RESTful API documentation by swagger and springboot
date: 2018-11-29
author: BF
header-img: img/bf/mountain_02.jpg
catalog: true
tags:
  - swagger
  - node.js
---

# 背景

利用 Swagger 官方的 Editor 和 UI 已经能够展示文档，但最后做为交付的文档，肯定是需要能直接打开的文档(e.g. html 格式)才行。所以这两天研究了之后，找到了几个解决方案

- [pretty-swag](https://github.com/twskj/pretty-swag)
- [spectacle](https://github.com/sourcey/spectacle)
- [bootprint-openapi](https://github.com/bootprint/bootprint-openapi)
- [html-inline](https://github.com/substack/html-inline) (把 html,js,css 整合到一个 html 中)
  他们都是基于 Node.js 的，下面分别来介绍下怎么使用。

# [pretty-swag](https://github.com/twskj/pretty-swag)

安装和使用都非常简单，安装完后直接运行，加载 swagger json 或者 yaml 文档，就能生成单个 html 文件。

下面是简单的使用方式，还有别的参数可以设置主题等，可以查看官方 github 得到更多信息：

```node.js
npm install pretty-swag -g
pretty-swag -i input.json -o output.html
```

最后的样子如下，完整示例在[这里](https://bearfly1990.github.io/apidoc/swagger/pretty-swagger.html)：
![pretty-swag](/img/post/2018/11/2018-11-29-SwaggerOnePage_01.jpg)

# (https://github.com/sourcey/spectacle)

spectacle 最后的样式我个人觉得还是不错的，不过貌似 leader 不是很喜欢。

```node.js
npm install -g spectacle-docs
spectacle -d input.json
```

运行了上述的命令后，会生成一个 public 文件夹，最后的文档就在其中：
![spectacle folder](/img/post/2018/11/2018-11-29-SwaggerOnePage_02.jpg)

最后的样子如下，完整示例在[这里](https://bearfly1990.github.io/apidoc/swagger/spectacle/)：
![pretty-swag](/img/post/2018/11/2018-11-29-SwaggerOnePage_03.jpg)

# [bootprint-openapi](https://github.com/bootprint/bootprint-openapi)

没想到最后领导还是喜欢这个样式，哈哈，主要就像他说的，他喜欢 model 的定义。

使用的方法如下：

```node.js
npm install -g bootprint
npm install -g bootprint-openapi
bootprint openapi input.json bootprint-openapi
```

运行了上述的命令后，会生成一个指定的文件夹，最后的文档就在其中：

![spectacle folder](/img/post/2018/11/2018-11-29-SwaggerOnePage_04.jpg)

最后的样子如下，完整示例在[这里](https://bearfly1990.github.io/apidoc/swagger/bootprint-openapi/)：
![pretty-swag](/img/post/2018/11/2018-11-29-SwaggerOnePage_05.jpg)

# [html-inline](https://github.com/substack/html-inline)

在 bootprint-openapi 项目里就有介绍，这个工具是把分开的 js,css 都合到一个 html 中。

使用方式如下：

```node.js
npm -g install html-inline
html-inline target/index.html
```

我在尝试的过程中发现，他目前只支持本地的引入，也就是说如果用的是在线的 js 库，比如像下面这样的，就不能正确生成。

如果要实现，我想自己应该可以去改一下源代码来实现，多一些判断与资源的获取。

```js
<script src="https://code.jquery.com/jquery-3.1.1.min.js" />
```

# 最后

其实 swagger ui 的样式也已经有办法能拿到，就是利用他自己本身的配置，将.json 或者.yaml 直接以对象的形式写进去。

使用的配置是 `spec`。

```js
var SwaggerUIBundle = require('swagger-ui-dist').SwaggerUIBundle

const ui = SwaggerUIBundle({
    //url: "https://petstore.swagger.io/v2/swagger.json",
    spec:{"swagger":"2.0","info":{"description":"This is a sample server Petstore server.  You can find out more about     Swagger at [http://swagger.io](http://swagger.io) or on [irc.freenode.net, #swagger](http://swagger.io/irc/).      For this sample, you can use the api key `special-key` to test the authorization     filters.","version":"1.0.0","title":"Swagger Petstore","termsOfService":"http://swagger.io/terms/","contact":{"email":"apiteam@swagger.io"},"license":{"name":"Apache 2.0","url":"http://www.apache.org/licenses/LICENSE-2.0.html"}},"host":"petstore.swagger.io","basePath":"/v2","tags":[{"name":"pet","description":"Everything about your Pets","externalDocs":{"description":"Find out more","url":"http://swagger.io"}},{"name":"store","description":"Access to Petstore orders"},{"name":"user","description":"Operations about user","externalDocs":{"description":"Find out more about our store","url":"http://swagger.io"}}],"schemes":["https","http"],"paths":{"/pet":{"post":{"tags":["pet"],"summary":"Add a new pet to the store","description":"","operationId":"addPet","consumes":["application/json","application/xml"],"produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"Pet object that needs to be added to the store","required":true,"schema":{"$ref":"#/definitions/Pet"}}],"responses":{"405":{"description":"Invalid input"}},"security":[{"petstore_auth":["write:pets","read:pets"]}]},"put":{"tags":["pet"],"summary":"Update an existing pet","description":"","operationId":"updatePet","consumes":["application/json","application/xml"],"produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"Pet object that needs to be added to the store","required":true,"schema":{"$ref":"#/definitions/Pet"}}],"responses":{"400":{"description":"Invalid ID supplied"},"404":{"description":"Pet not found"},"405":{"description":"Validation exception"}},"security":[{"petstore_auth":["write:pets","read:pets"]}]}},"/pet/findByStatus":{"get":{"tags":["pet"],"summary":"Finds Pets by status","description":"Multiple status values can be provided with comma separated strings","operationId":"findPetsByStatus","produces":["application/xml","application/json"],"parameters":[{"name":"status","in":"query","description":"Status values that need to be considered for filter","required":true,"type":"array","items":{"type":"string","enum":["available","pending","sold"],"default":"available"},"collectionFormat":"multi"}],"responses":{"200":{"description":"successful operation","schema":{"type":"array","items":{"$ref":"#/definitions/Pet"}}},"400":{"description":"Invalid status value"}},"security":[{"petstore_auth":["write:pets","read:pets"]}]}},"/pet/findByTags":{"get":{"tags":["pet"],"summary":"Finds Pets by tags","description":"Muliple tags can be provided with comma separated strings. Use         tag1, tag2, tag3 for testing.","operationId":"findPetsByTags","produces":["application/xml","application/json"],"parameters":[{"name":"tags","in":"query","description":"Tags to filter by","required":true,"type":"array","items":{"type":"string"},"collectionFormat":"multi"}],"responses":{"200":{"description":"successful operation","schema":{"type":"array","items":{"$ref":"#/definitions/Pet"}}},"400":{"description":"Invalid tag value"}},"security":[{"petstore_auth":["write:pets","read:pets"]}],"deprecated":true}},"/pet/{petId}":{"get":{"tags":["pet"],"summary":"Find pet by ID","description":"Returns a single pet","operationId":"getPetById","produces":["application/xml","application/json"],"parameters":[{"name":"petId","in":"path","description":"ID of pet to return","required":true,"type":"integer","format":"int64"}],"responses":{"200":{"description":"successful operation","schema":{"$ref":"#/definitions/Pet"}},"400":{"description":"Invalid ID supplied"},"404":{"description":"Pet not found"}},"security":[{"api_key":[]}]},"post":{"tags":["pet"],"summary":"Updates a pet in the store with form data","description":"","operationId":"updatePetWithForm","consumes":["application/x-www-form-urlencoded"],"produces":["application/xml","application/json"],"parameters":[{"name":"petId","in":"path","description":"ID of pet that needs to be updated","required":true,"type":"integer","format":"int64"},{"name":"name","in":"formData","description":"Updated name of the pet","required":false,"type":"string"},{"name":"status","in":"formData","description":"Updated status of the pet","required":false,"type":"string"}],"responses":{"405":{"description":"Invalid input"}},"security":[{"petstore_auth":["write:pets","read:pets"]}]},"delete":{"tags":["pet"],"summary":"Deletes a pet","description":"","operationId":"deletePet","produces":["application/xml","application/json"],"parameters":[{"name":"api_key","in":"header","required":false,"type":"string"},{"name":"petId","in":"path","description":"Pet id to delete","required":true,"type":"integer","format":"int64"}],"responses":{"400":{"description":"Invalid ID supplied"},"404":{"description":"Pet not found"}},"security":[{"petstore_auth":["write:pets","read:pets"]}]}},"/pet/{petId}/uploadImage":{"post":{"tags":["pet"],"summary":"uploads an image","description":"","operationId":"uploadFile","consumes":["multipart/form-data"],"produces":["application/json"],"parameters":[{"name":"petId","in":"path","description":"ID of pet to update","required":true,"type":"integer","format":"int64"},{"name":"additionalMetadata","in":"formData","description":"Additional data to pass to server","required":false,"type":"string"},{"name":"file","in":"formData","description":"file to upload","required":false,"type":"file"}],"responses":{"200":{"description":"successful operation","schema":{"$ref":"#/definitions/ApiResponse"}}},"security":[{"petstore_auth":["write:pets","read:pets"]}]}},"/store/inventory":{"get":{"tags":["store"],"summary":"Returns pet inventories by status","description":"Returns a map of status codes to quantities","operationId":"getInventory","produces":["application/json"],"parameters":[],"responses":{"200":{"description":"successful operation","schema":{"type":"object","additionalProperties":{"type":"integer","format":"int32"}}}},"security":[{"api_key":[]}]}},"/store/order":{"post":{"tags":["store"],"summary":"Place an order for a pet","description":"","operationId":"placeOrder","produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"order placed for purchasing the pet","required":true,"schema":{"$ref":"#/definitions/Order"}}],"responses":{"200":{"description":"successful operation","schema":{"$ref":"#/definitions/Order"}},"400":{"description":"Invalid Order"}}}},"/store/order/{orderId}":{"get":{"tags":["store"],"summary":"Find purchase order by ID","description":"For valid response try integer IDs with value >= 1 and <= 10.         Other values will generated exceptions","operationId":"getOrderById","produces":["application/xml","application/json"],"parameters":[{"name":"orderId","in":"path","description":"ID of pet that needs to be fetched","required":true,"type":"integer","maximum":10,"minimum":1,"format":"int64"}],"responses":{"200":{"description":"successful operation","schema":{"$ref":"#/definitions/Order"}},"400":{"description":"Invalid ID supplied"},"404":{"description":"Order not found"}}},"delete":{"tags":["store"],"summary":"Delete purchase order by ID","description":"For valid response try integer IDs with positive integer value.         Negative or non-integer values will generate API errors","operationId":"deleteOrder","produces":["application/xml","application/json"],"parameters":[{"name":"orderId","in":"path","description":"ID of the order that needs to be deleted","required":true,"type":"integer","minimum":1,"format":"int64"}],"responses":{"400":{"description":"Invalid ID supplied"},"404":{"description":"Order not found"}}}},"/user":{"post":{"tags":["user"],"summary":"Create user","description":"This can only be done by the logged in user.","operationId":"createUser","produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"Created user object","required":true,"schema":{"$ref":"#/definitions/User"}}],"responses":{"default":{"description":"successful operation"}}}},"/user/createWithArray":{"post":{"tags":["user"],"summary":"Creates list of users with given input array","description":"","operationId":"createUsersWithArrayInput","produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"List of user object","required":true,"schema":{"type":"array","items":{"$ref":"#/definitions/User"}}}],"responses":{"default":{"description":"successful operation"}}}},"/user/createWithList":{"post":{"tags":["user"],"summary":"Creates list of users with given input array","description":"","operationId":"createUsersWithListInput","produces":["application/xml","application/json"],"parameters":[{"in":"body","name":"body","description":"List of user object","required":true,"schema":{"type":"array","items":{"$ref":"#/definitions/User"}}}],"responses":{"default":{"description":"successful operation"}}}},"/user/login":{"get":{"tags":["user"],"summary":"Logs user into the system","description":"","operationId":"loginUser","produces":["application/xml","application/json"],"parameters":[{"name":"username","in":"query","description":"The user name for login","required":true,"type":"string"},{"name":"password","in":"query","description":"The password for login in clear text","required":true,"type":"string"}],"responses":{"200":{"description":"successful operation","schema":{"type":"string"},"headers":{"X-Rate-Limit":{"type":"integer","format":"int32","description":"calls per hour allowed by the user"},"X-Expires-After":{"type":"string","format":"date-time","description":"date in UTC when token expires"}}},"400":{"description":"Invalid username/password supplied"}}}},"/user/logout":{"get":{"tags":["user"],"summary":"Logs out current logged in user session","description":"","operationId":"logoutUser","produces":["application/xml","application/json"],"parameters":[],"responses":{"default":{"description":"successful operation"}}}},"/user/{username}":{"get":{"tags":["user"],"summary":"Get user by user name","description":"","operationId":"getUserByName","produces":["application/xml","application/json"],"parameters":[{"name":"username","in":"path","description":"The name that needs to be fetched. Use user1 for testing. ","required":true,"type":"string"}],"responses":{"200":{"description":"successful operation","schema":{"$ref":"#/definitions/User"}},"400":{"description":"Invalid username supplied"},"404":{"description":"User not found"}}},"put":{"tags":["user"],"summary":"Updated user","description":"This can only be done by the logged in user.","operationId":"updateUser","produces":["application/xml","application/json"],"parameters":[{"name":"username","in":"path","description":"name that need to be updated","required":true,"type":"string"},{"in":"body","name":"body","description":"Updated user object","required":true,"schema":{"$ref":"#/definitions/User"}}],"responses":{"400":{"description":"Invalid user supplied"},"404":{"description":"User not found"}}},"delete":{"tags":["user"],"summary":"Delete user","description":"This can only be done by the logged in user.","operationId":"deleteUser","produces":["application/xml","application/json"],"parameters":[{"name":"username","in":"path","description":"The name that needs to be deleted","required":true,"type":"string"}],"responses":{"400":{"description":"Invalid username supplied"},"404":{"description":"User not found"}}}}},"securityDefinitions":{"petstore_auth":{"type":"oauth2","authorizationUrl":"http://petstore.swagger.io/oauth/dialog","flow":"implicit","scopes":{"write:pets":"modify pets in your account","read:pets":"read your pets"}},"api_key":{"type":"apiKey","name":"api_key","in":"header"}},"definitions":{"Order":{"type":"object","properties":{"id":{"type":"integer","format":"int64"},"petId":{"type":"integer","format":"int64"},"quantity":{"type":"integer","format":"int32"},"shipDate":{"type":"string","format":"date-time"},"status":{"type":"string","description":"Order Status","enum":["placed","approved","delivered"]},"complete":{"type":"boolean","default":false}},"xml":{"name":"Order"}},"Category":{"type":"object","properties":{"id":{"type":"integer","format":"int64"},"name":{"type":"string"}},"xml":{"name":"Category"}},"User":{"type":"object","properties":{"id":{"type":"integer","format":"int64"},"username":{"type":"string"},"firstName":{"type":"string"},"lastName":{"type":"string"},"email":{"type":"string"},"password":{"type":"string"},"phone":{"type":"string"},"userStatus":{"type":"integer","format":"int32","description":"User Status"}},"xml":{"name":"User"}},"Tag":{"type":"object","properties":{"id":{"type":"integer","format":"int64"},"name":{"type":"string"}},"xml":{"name":"Tag"}},"Pet":{"type":"object","required":["name","photoUrls"],"properties":{"id":{"type":"integer","format":"int64"},"category":{"$ref":"#/definitions/Category"},"name":{"type":"string","example":"doggie"},"photoUrls":{"type":"array","xml":{"name":"photoUrl","wrapped":true},"items":{"type":"string"}},"tags":{"type":"array","xml":{"name":"tag","wrapped":true},"items":{"$ref":"#/definitions/Tag"}},"status":{"type":"string","description":"pet status in the store","enum":["available","pending","sold"]}},"xml":{"name":"Pet"}},"ApiResponse":{"type":"object","properties":{"code":{"type":"integer","format":"int32"},"type":{"type":"string"},"message":{"type":"string"}}}},"externalDocs":{"description":"Find out more about Swagger","url":"http://swagger.io"}}
    dom_id: '#swagger-ui',
    presets: [
      SwaggerUIBundle.presets.apis,
      SwaggerUIBundle.SwaggerUIStandalonePreset
    ],
    layout: "StandaloneLayout"
  })

```

能想到的暂时就这几个办法，有发现更好的就继续更新。

---

参考：

- [Writing OpenAPI (Swagger) Specification Tutorial](https://apihandyman.io/writing-openapi-swagger-specification-tutorial-part-1-introduction/)
- [Swagger 从入门到精通](https://www.gitbook.com/book/huangwenchao/swagger/details)
- [OpenAPI Specification 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md)
- [Converting Swagger specification JSON to HTML documentation](https://stackoverflow.com/questions/25800493/converting-swagger-specification-json-to-html-documentation)
