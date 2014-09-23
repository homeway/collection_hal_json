在HAL中使用集合
=============

相关链接：

* [HAL语义说明](http://stateless.co/hal_specification.html)
* [HAL开发包](https://github.com/mikekelly/hal_specification/wiki/Libraries)

在不破坏HAL优雅结构的情况下，我定义了一种通用的profile来表达集合的语义。
<br/>使用这些关键字，最简单的方法是使用`curies`语法。

例子
---
###获取一个集合
```json
GET "/customers"
{
  "_links":{
    "self": {"href": "/customers"},
    "curies": [{ 
      "name": "m", 
      "href": "http://hotmoon.org/docs/rels/collection#{rel}", 
      "templated": true }]},
  "m:schema": "collection",
  "m:fields": {
    "create": {
      "name": {"type":"text"}, 
      "email": {"type":"email"}}
  },
  "m:actions": {"create":{}},
  "_embedded":{
    "m:items":[]
  }
}
```
###提交创建资源的请求
```json
POST "/customers"
{
  "name": "adi", 
  "email": "adi@gmail.com"
}
```
###获取单个资源
```json
GET "/customers/1"
{
  "_links":{
    "self": {"href": "/customers/1"},
    "curies": [{ 
      "name": "m", 
      "href": "http://hotmoon.org/docs/rels/collection#{rel}", 
      "templated": true }]},
  "m:schema": "resource",
  "m:fields": {
    "update": {
      "name": {"type":"text"}, 
      "email": {"type":"email"}}
  },
  "m:actions": {"delete":{}, "update":{}},
  "m:properties":{
    "name": "adi", 
    "email": "adi@gmail.com"}
}
```

关键字定义
--------
* schema
* items
* properties
* actions
* fields

schema
------
`schema`的取值为"collection"或"resource"，分别表示集合或单个资源。
<br/>这里强制要求了资源表述中不应该同时包含集合模型和单个资源模型。

###collection
集合模型。
<br>默认的集合模型，提供了集合的列表查询、向集合中添加新的资源等方法：

* self
* create

###resource
单个资源模型。

默认的单个资源模型，提供了资源的查询和删除等方法：

* self
* update
* delete

items
-----
如果选择了`collection`模式，意味着`_embedded`中应包含的`items`来表示资源列表。

```json
{
  ...
  "_embedded":{
    "m:items":[
      {
        "_links":{"self": { "href": "/customers/101" }},
        "m:schema": "resource",
        "m:properties":{
          "id": 101,
          "name": "adi", 
          "email": "adi@gmail.com"}},
      {
        "_links":{"self": { "href": "/customers/102" }},
        "m:schema": "resource",
        "m:properties":{
          "id": 102,
          "name": "adi", 
          "email": "adi@gmail.com"}
      },
      ...
    ]
  }
  ...
}
```

请注意，上面的例子中，被嵌入的资源除了已经指定的`self`链接关系还包含了两个默认的链接关系：`update`、`delete`。

properties
----------
如果选择了`resource`，则应提供`properties`来描述资源本身的属性。

```json
{
  ...
  "m:properties":{
    "id": 102,
    "name": "adi", 
    "email": "adi@gmail.com"}
}
```

actions
-------
如果希望客户端自动发现可用的操作，可以定义这个选项。
<br/>这比协议语义上的`OPTIONS`方法有更具体的应用语义。
<br/>服务端在提供这个选项时还可以建立在授权机制的基础上。

```json
{
  "{action}": {
    "href": "resource_uir",
    "method": "http_verb",
    "title": "a title"}
}
```

###action
为动作起的名字，例如："create"，就是一个保留的名字

下面是预定义的action：

* create，向集合内添加新的资源。默认为`collection`所有。
* update，修改资源。默认为`resource`所有。
* delete，删除资源。默认为`resource`所有。

按照默认的规则，上面集合示例中的`actions`部分是可以省略的。

下面的例子对集合作了只读配置：

```json
{
  ...
  "m:actions": {}
}
```

下面的例子说明，集合中还有两个额外的action：

```json
{
  ...
  "m:actions":{
    "modify_password": {
      "href": "/customers/102/modify_password",
      "method": "patch"},
    "modify_profile": {
      "href": "/customers/102/modify_profile",
      "method": "patch"}}
}
```

###href
资源转移的URL。默认为self中指定的方法。

###method
HTTP方法。在已知方法中是可选的，如果是新方法则必须指定。

###title
链接标题。可选。

fields
------
字段描述，很多时候是为了描述表单。如果存在http的动作为`post`、`put`或`patch`等不安全的方法，则需要实现这个字段，否则客户端将无法自动判断需要填写的表单类型。

```json
{
  "{action_name}": {
    "{filed}": {"type":"", "title":"", "value":""}, 
    "{filed}": {"type":"", "title":"", "value":""}, 
    ...
    }
}
```

###{action_name}
表单对应的同名动作名称，如`create`, `update`。

###{filed}
用户根据业务定义的、机器可读的字段名。

###type
字段类型。可选。默认就是`text`类型。

可以HTML5中已经实现的input任意类型（这里实际上借鉴了siren的做法）：

`hidden`, `text`, `search`, `tel`, `url`, `email`, `password`, `datetime`, `date`, `month`, `week`, `time`, `datetime-local`, `number`, `range`, `color`, `checkbox`, `radio`, `file`, `image`, `button`

###name
机器可读的名字。必选。

###title
人类可读的标题。可选。

###value
默认值。可选。

高级主题
-------

###表单校验
可扩展`fields`描述，使用HTML5的惯用的表单校验方法。

###集合或资源中的`self`自描述
实际上，HAL中指定的链接关系`self`也被扩展了，可以通过`m:fields:{"self":{...}}`来描述应该如何展现资源，这同样利于编写智能的客户端代码。

```json
  {
    ...
    "m:fields": {
      "self": {
        "id": {"title": "ID"},
        "name": {"title":"名称"}, 
        "email": {"title":"邮件", "type":"email"}}
    },
  }
```

###直接支持集合的创建和删除
虽然这不是通常的做法，但你有时也许确实可以考虑。
<br/>下面是一个例子，支持直接创建一个集合和删除一个集合。
<br/>这个例子还示范了如何增加自定义方法。
```json
{
  ...
  "m:actions": {
    "create_cagtegory":{
      "href":"/customers/{category}", 
      "templated":true, 
      "method":"put"},
    "delete_cagtegory":{
      "href":"/customers/{category}", 
      "templated":true, 
      "method":"delete"}}
  ...
}    
```


