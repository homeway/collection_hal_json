JSON Hypertext Application Language
===================================
作者： M.Kelly, Stateless
<br/>2013年10月3日提交的RFC草案，但2014年4月6日已过期

翻译：homeway.xue@gmail.com

介绍
---
很多没有HTML的应用都提供基于Web的API，使用超链接让资源直接与客户端通信。

HAL非常善于表达超链接控件，例如JSON[RFC4627](http://tools.ietf.org/pdf/rfc4627)的链接。

HAL是可以让Web的API开发和提供一系列链接的通用媒体类型。API客户端可以通过链接关系选择链接，并处理应用程序的导航。

HAL使用超媒体提供统一的接口语义，使创建可重用的HAL开发工具包成为可能。

HAL的主要设计目标是通用和简洁。HAL可以被应用在很多领域，并提供了一个超媒体Web的API设计的最关键部分。

先决条件
------
文档中的`MUST`，`MUST NOT`，`REQRUIED`, `SHALL`, `SHALL NOT`, `SHOULD`, `SHOULD NOT`, `RECOMMANDED`, `MAY`以及`OPTIONAL`等关键字在[RFC2119](http://tools.ietf.org/pdf/rfc2119)中描述。

HAL文档
------
一个HAL文档格式在[RFC4627](http://tools.ietf.org/pdf/rfc4627)中描述，并有一个`application/hal+json`媒体类型。

根对象必须是一个资源对象。

例如：

    GET /orders/523 HTTP/1.1
    Host: example.org
    Accept: application/hal+json
    HTTP/1.1 200 OK
    Content-Type: application/hal+json
```json

{
"_links": {
     "self": { "href": "/orders/523" },
     "warehouse": { "href": "/warehouse/56" },
     "invoice": { "href": "/invoices/873" }
   },
   "currency": "USD",
   "status": "shipped",
   "total": 10.20
}
```
这里，我们使用URI"/order/523"来获得文档和一个order资源，它包含了"warehourse"和"invoice"链接，它还在自己的表单状态中提供"currency"，"status"和"total"属性。

资源对象
-------
一个资源对象表示一个资源。它具有两个保留的属性：

(1) "_links": 包含其他资源的链接
(2) "_embedded": 包含嵌入的资源

所有其他属性必须是合法的JSON，并表达了资源的当前状态。

###保留的属性
#### _links
保留字`_links`是可选的。

这个对象拥有若干属性，名字是链接类型[RFC5988](http://tools.ietf.org/pdf/rfc5988)，值是链接对象或链接对象的数组。包含"_links"对象的资源的才是主题资源。

#### _embedded
保留字`_embedded`是可选的。

这个对象拥有若干属性，名字是链接类型[RFC5988](http://tools.ietf.org/pdf/rfc5988)，值是链接对象或链接对象的数组。

目标URI提供的嵌入资源可以是完整的表述，也可以是部分表述，或干脆是另外版本的表述。

链接对象
-------
一个链接对象表达一个从资源到URI的超媒体控件。它具有下列属性：

### href
"href"属性是必须的。

它既可以是一个URI[RFC3986](http://tools.ietf.org/pdf/rfc3986)，也可以是一个URI模板[RFC6570](http://tools.ietf.org/pdf/rfc6570)。

如果值是一个URI模板，那么链接对象应该提供一个"templated"为true的属性值。

### templated
"templated"属性是可选的。

它的值是布尔类型。如果链接对象的"href"属性是一个URI模板，那就应该将其设置为ture。

如果没有定义或设成了其他任何非true的值，都应当作false来处理.

### type
"type"属性是可选的。

它的值是用来提示或指引媒体类型的一个字符串，当需要解析目标资源时会很有用。

### deprecation
"deperacation"属性是可选的。

### name
### profile
### title
### hreflang

示例文档
-------
下面是一个表达一个订单列表的示例文档

    GET /orders HTTP/1.1
    Host: example.org
    Accept: application/hal+json
    HTTP/1.1 200 OK
    Content-Type: application/hal+json
```json
{
  "_links": {
    "self": { "href": "/orders" },
    "next": { "href": "/orders?page=2" },
    "find": { "href": "/orders{?id}", "templated": true }
  },
  "_embedded": {
  "orders": [{
    "_links": {
        "self": { "href": "/orders/123" },
        "basket": { "href": "/baskets/98712" },
        "customer": { "href": "/customers/7809" }},
      "total": 30.00,
      "currency": "USD",
      "status": "shipped",
    },{
      "_links": {
        "self": { "href": "/orders/124" },
        "basket": { "href": "/baskets/97213" },
        "customer": { "href": "/customers/12369" }},
      "total": 20.00,
      "currency": "USD",
      "status": "processing"
    }]
  },
  "currentlyProcessing": 14,
  "shippedToday": 20
}    
```
媒体类型参数
----------
### profile

建议
---
### self链接

### 链接关系

### 超文本缓存模式

加密考虑
-------

IANA考虑
-------

规范参考
-------

附录A 致谢
---------

附录B FAQ
---------