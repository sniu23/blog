# OpenAPI规范 v2.0 版本

> 前身：Swagger RESTful API 文档规范，简称：OAS

[github地址](https://swagger.io/specification/v2/) | 
[swagger地址](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) | 
[中文资料](https://huangwenchao.gitbooks.io/swagger/content/)

# 一、规范

- `路径模板 Path Templating`： 使用一对大括号 {} 标记 URL 中可以被参数替换的部分。
- `媒体类型 Mime Types`： 遵循 RFC 6838。
- `HTTP状态码 HTTP Status Codes`： 遵循 RFC 7231。
- `格式`： 支持 JSON 和 YAML 格式。
- `字段名称`： 区分大小写； 可以固定命名，也可以使用正则表达式匹配。
- `文件`： 由单一文件构成（允许通过`$ref`字段拆分到多个文件中），通常文件命名为`swaager.json`。
- 规范中某些对象可以声明保持为空，或者完全删除。（允许在文档上附加ACL“访问控制层”，以某种形式的认证/授权来选择允许访问的文档的部分。）

# 二、简单数据类型：

- `原始数据类型`基于 JSON-Schema Draft 4 支持
  - 属性`format` 用于更细致的定义描述。（开放型 string 类型，如：email， uuid）
- 一个附加的数据类型： `file` 仅在 `Parameter Object` 或 `Response Object` 中使用。
- `复杂数据类型`： `Info Object`, `Paths Object`, `Parameters Object`, `Responses Definitions Object`……

|通用名字|`类型`|`格式`|备注|
|-|-|-|-|
|integer |integer |int32 |signed 32位|
|long |integer |int64 |signed 64位|
|float |number |float |-|
|double |number |double |-|
|string |string | |-|
|byte |string |byte |base64编码字符|
|binary |string |binary |-|
|boolean |boolean | |-|
|date |string |date |RFC3339定义: "full-date"，如：'2016-04-01' |
|dateTime |string |date-time |RFC3339定义: "date-time", 如：'2016-04-16T16:06:05Z' |
|password |string |password |用来提示UI输入时遮蔽 |

# 三、根结构：

|字段名称|类型|描述|
|-|-|-|
|swagger |string |**必填**，Swagger规范版本，**必须是“2.0”** |
|info |`Info Object` |**必填**，提供API相关元数据。 |
|paths |`Paths Object` |**必填**，可以获得的paths和操作。 |
|host |string |host地址：域名或ip，可以包含端口，但不得包含其他路径 |
|basePath |string |必须以“/”开头，定义基础路径。 |
|schemes |[string] |传输协议：`http,https,ws,wss`。不指定时默认是访问swagger的协议 |
|consumes |[string] |一系列被消费的MIME类型 |
|produces |[string] |一系列被生产的MIME类型 |
|definitions |`Definitions Object` |定义生产和消费中操作的数据类型 |
|parameters |`Parameters Definitions Object` |操作中使用的参数 |
|responses |`Responses Definitions Object` |操作的结果 |
|securityDefinitions |`Security Definitions Object` |定义安全策略schema |
|security |[`Security Requirement Object`] |声明一系列应用的安全策略 |
|tags |[`Tag Object`] |一系列唯一标签 |
|externalDocs |`External Documentation object` |外部文档 |
|^x- |any |swagger允许的拓展 |

```code
# MIME类型实例
  text/plain; charset=utf-8
  application/json
  application/vnd.github+json
  application/vnd.github.v3+json
  application/vnd.github.v3.raw+json
  application/vnd.github.v3.text+json
  application/vnd.github.v3.html+json
  application/vnd.github.v3.full+json
  application/vnd.github.v3.diff
  application/vnd.github.v3.patch
```

# 四、API信息描述、标签定义、外部文档定义：

```yaml
# Info Object
title: Swagger Sample App ### 必填！！！
description: This is a sample server Petstore server. # 支持GFM语法(https://guides.github.com/features/mastering-markdown/#GitHub-flavored-markdown)
termsOfService: http://swagger.io/terms/ # 服务协议
contact: # + Contact Object
  name: API Support
  url: http://www.swagger.io/support
  email: support@swagger.io
license: # + License Object 遵循服务
  name: Apache 2.0 ### 必填！！！
  url: http://www.apache.org/licenses/LICENSE-2.0.html
version: 1.0.1 ### 必填！！！
```

```yaml
# Tag Object
name: pet ### 必填！！！
description: Pets operations
externalDocs: # External Documentation object
```

```yaml
# External Documentation object
description: Find more info here
url: https://swagger.io ### 必填！！！
```

```yaml
# Reference Object
$ref: '#/definitions/Pet'

$ref: 'Pet.yaml'

$ref: 'definitions.yaml#/Pet'
```

```yaml
# Example Object
application/json: # MINE TYPE
  name: Puma
  type: Dog
  color: Black
  gender: Female
  breed: Mixed
```

```yaml
# Xml Object

```

# 五、模型对象 Schema Object 

- 通用字段有：
  - `description`
  - `default`： 默认值，如请求中每页的条目数。
  - `enum`
  - `type`： 'array', 'string', 'number', 'integer', 'boolean', 'file', 'object'
  - `$ref`
  - `allOf`： **组合**一组对象，形成新的对象。
  - `title`
  - `discriminator`： **多态**性支持：如果其他schema继承本schema，定义一组属性，用来区分。
  - `readOnly`
  - `xml`： XML Object
  - `externalDocs`： External Documentation object
  - `example`： 任意值，样例
- `type` 是 `array`：
  - `items`
  - `maxItems`： 数值中的最大元素个数 integer
  - `minItems`： 数值中的最小元素个数 integer
  - `uniqueItems`： 标识数组中的元素是否唯一 boolean
  - `collectionFormat`: 
    - "csv", 默认，逗号分割，如：foo,bar
    - "ssv", 空格分割，如：foo bar
    - "tsv", 反斜杠分割，如：foo\bar
    - "pipes", 竖线分割，如：foo|bar
    - "multi", 单属性可以取多个值（适用查询参数和表单参数），如：foo=bar&foo=baz
- `type` 是 `string`：
  - `format`： type的扩展格式 string
  - `pattern`
  - `maxLength`： 字符串最大长度
  - `minLength`： 字符串最小长度
- `type` 是 `number` 或 `integer`：
  - `format`
  - `maximum`： 最大值 number
  - `exclusiveMaximum`： 数值是否必须小于最大值 boolean
  - `minimum`： 最小值 number
  - `exclusiveMinimum`： 数值是否必须大于最小值 boolean
  - `multipleOf`： 数值必须是multipleOf的整数倍 number
- `type` 是 `object`：
  - `properties`
  - `additionalProperties`
  - `required`
  - `maxProperties`
  - `minProperties`
- `type` 是 `file`：
  - `maxLength`
  - `minLength`

```yaml
# 定义支持原生类型
type: string
format: email
```

```yaml
# 定义一个简单模型
type: object
required:
- name
properties:
  name:
    type: string
  address:
    $ref: '#/definitions/Address'
  age:
    type: integer
    format: int32
    minimum: 0
```

```yaml
# 定义一个带映射属性的模型
type: object
additionalProperties:
  type: string
```

```yaml
# 定义一个带映射属性(映射至另一个模型)的模型
type: object
additionalProperties:
  $ref: '#/definitions/ComplexModel'
```

```yaml
# 定义一个带样例的模型
type: object
properties:
  id:
    type: integer
    format: int64
  name:
    type: string
required:
- name
example:
  name: Puma
  id: 1
```

```yaml
# 一个模型使用 allOf “组合”另外一个模型
definitions:
  ErrorModel:
    type: object
    required:
    - message
    - code
    properties:
      message:
        type: string
      code:
        type: integer
        minimum: 100
        maximum: 600
  ExtendedErrorModel:
    allOf:
    - $ref: '#/definitions/ErrorModel'
    - type: object
      required:
      - rootCause
      properties:
        rootCause:
          type: string
```

```yaml
# 使用 discriminator 声明必须“继承”的字段
definitions:
  Pet:
    type: object
    discriminator: petType
    properties:
      name:
        type: string
      petType:
        type: string
    required:
    - name
    - petType
  Cat:
    description: A representation of a cat
    allOf:
    - $ref: '#/definitions/Pet'
    - type: object
      properties:
        huntingSkill:
          type: string
          description: The measured skill for hunting
          default: lazy
          enum:
          - clueless
          - lazy
          - adventurous
          - aggressive
      required:
      - huntingSkill
  Dog:
    description: A representation of a dog
    allOf:
    - $ref: '#/definitions/Pet'
    - type: object
      properties:
        packSize:
          type: integer
          format: int32
          description: the size of the pack the dog is from
          default: 0
          minimum: 0
      required:
      - packSize
```

`Schema Object` 中的很多字段在其他对象也有出现：`Parameter Object`，`Header Object`，`Items Object`

# 六、接口定义层次

- `Paths` (接口：路径+操作)
  - `Path` 路径
    - `Operation` 操作
      - `Parameters` 参数
      - `Responses` 响应 
    - `Parameters`
- ...
- Definitions
- Parameters
- Responses
- Security Definitions
- Security

# 七、Operation 操作

允许的HTTP谓词有： `get`，`post`，`put`，`head`，`delete`，`options`， `patch`.

```yaml
paths: # Paths Object 
  /pet/{petId}: # Path Item Object
    post: # Operation Object 
      tags: # [string]
      - pet
      summary: Updates a pet in the store with form data ### 限定120个字符内！！
      description: ""
      externalDocs: # External Documentation Object
      operationId: updatePetWithForm # 操作对象 Operation Object 唯一ID
      consumes: # 被消费的MIME类型
      - application/x-www-form-urlencoded
      produces: # 被生产的MIME类型
      - application/json
      - application/xml
      parameters: # [Parameter Object | Reference Object]
      - name: petId
        in: path
        description: ID of pet that needs to be updated
        required: true
        type: string
      - name: name
        in: formData
        description: Updated name of the pet
        required: false
        type: string
      - name: status
        in: formData
        description: Updated status of the pet
        required: false
        type: string
      responses: # Responses Object ### 必填！！！
        '200':
          description: Pet updated.
        '405':
          description: Invalid input
      schemes: # 传输协议 ["http","https","ws","wss"]
      deprecated: # 申明操作是否已废弃。默认：false
      security: # 安全策略 [Security Requirement Object]
      - petstore_auth:
        - write:pets
        - read:pets
```

# 八、Parameters 参数

5种参数类型： `path`, `query`, `header`, `body`, `formData`。
  - `body`和`formData`都属于附加在HTTP请求上的payload，而负载只能有一个。所以：
    - `body`和`formData`不能同时使用。
    - `body`参数只能有一个（参数名没有实际意义）！
  - `formData`是唯一支持文件传输的类型。
  - `formData`类型的请求的文档类型 `content-type`必须是：
    - `application/x-www-form-urlencoded`： 通常用于简单参数的传输。
      - （格式类似`query`，如：foo=1&bar=swagger）
    - `multipart/form-data`： 多用于文件传输。
      - （每个参数在payload中携带一个内部header，如：Content-Disposition: form-data; name="submit-name"）

字段：
+ 固定字段：
  - `name`： **必填**
  - `in`： **必填**, 参数的位置： `query`, `header`, `path`, `formData`, `body`
  - `required`： **必填**， 参数是否必填（`path`类型一定为 true）
  - `description`
+ `body`类型的字段：
  - `schema`： **必填**， 类型：schema object
+ 非`body`类型的字段：
  - `type`： **必填**， 只能是简单类型，可以是`file`，不能是`object`！！！
  - `allowEmptyValue`： 指定参数是否可以为空（仅对`query`和`formData`类型有效）
  - 其他字段参考：schema object

```yaml
# 一个带引用的 Body参数
name: user ### 必填！！！
in: body ### 必填！！！
description: user to add to the system
required: true 
schema: # Schema Object ### in:body必填！！！ 
  $ref: '#/definitions/User'

# 一个参数为string array的 Body参数
name: user ### 必填！！！
in: body ### 必填！！！
description: user to add to the system
required: true
schema: # Schema Object ### in:body 必填！！！
  type: array
  items:
    type: string
```

```yaml
# 一个64位整数 Header参数
name: token ### 必填！！！
in: header ### 必填！！！
description: token to be passed as a header
required: true
type: array
items: ### type: array 必填！！！
  type: integer
  format: int64
collectionFormat: csv
```

```yaml
# 一个string类型的 Path参数
name: username ### 必填！！！
in: path ### 必填！！！
description: username to fetch
required: true ### in:path 必填，且必须是true！！！
type: string
```

```yaml
# 一个可选的Query参数，string类型，可通过重复给出该查询参数接收多个值
name: id ### 必填！！！
in: query ### 必填！！！
description: ID of the object to fetch
required: false
type: array
items: 
  type: string
collectionFormat: multi
```

```yaml
# 一个带有文件类型的 Form数据，用于上传
name: avatar ### 必填！！！
in: formData ### 必填！！！
description: The avatar of the user
required: true
type: file
```

# 九、Responses 响应

- 至少包含一条响应代码（一般是成功操作响应）
- 不一定包含所有HTTP响应代码，可以包含：成功响应和已知异常响应。
- `default`被用作默认响应对象，规范未被覆盖的HTTP响应代码。

```yaml
# Responses Object
'200': # Response Object # HTTP status code 
  description: a pet to be returned ### 必填！！！
  schema: # Schema Object
    $ref: '#/definitions/Pet'
  headers: # Headers Object 
  examples: # Example Object
default:
  description: Unexpected error ### 必填！！！
  schema:
    $ref: '#/definitions/ErrorModel'
```

`Headers Object` 字段参考 `Schema Object` (其中`type`只能是：'array', 'string', 'number', 'integer', 'boolean'）

```yaml
# 响应返回一组复杂类型数组
description: A complex object array response
schema:
  type: array
  items:
    $ref: '#/definitions/VeryComplexType'

# 响应返回一个字符串类型
description: A simple string response
schema:
  type: string

# 响应携带Headers 
description: A simple string response
schema:
  type: string
headers: # Headers Object
  X-Rate-Limit-Limit:
    description: The number of allowed requests in the current period
    type: integer
  X-Rate-Limit-Remaining:
    description: The number of remaining requests in the current period
    type: integer
  X-Rate-Limit-Reset:
    description: The number of seconds left in the current period
    type: integer

# 响应无返回值
description: object created
```

# 十、Definitions 对象定义

Definitions Object 数据类型为： `Schema Object`，用来定义原始数据类型，数组或对象模型。

通过在`对象定义`、`参数定义`、`响应消息定义`中预定义后，使用 `$ref` 重复引用（重用）。
- 可以通过引用一个对象的定义来定义另一个对象。
- 不仅仅可以定义`全局性`参数，同样可以定义`路径级`共用的参数。

```yaml
definitions:
  Person:
    required:
      - username
    properties:
      firstName:
        type: string
      lastName:
        type: string
      username:
        type: string
  Persons:
    type: array
    items:
      $ref: "#/definitions/Person"
  Error:
    required:
      - code
      - message
    properties:
      code:
        type: string
      message:
        type: string

responses:
  Standard500ErrorResponse:
    description: An unexpected error occured.
    schema:
      $ref: "#/definitions/Error"
  PersonDoesNotExistResponse:
    description: Person does not exist.

parameters:
  username:
    name: username
    in: path
    required: true
    description: The person's username
    type: string
  pageSize:
    name: pageSize
    in: query
    description: Number of persons returned
    type: integer
  pageNumber:
    name: pageNumber
    in: query
    description: Page number
    type: integer
```

```yaml
# 通过 allOf 组合 两个对象的属性，形成新的对象

# PagedPersons 根节点下，具有将 Persons 和 Paging 展开 后的全部属性
  PagedPersons:
    allOf:
      - $ref: "#/definitions/Persons"
      - $ref: "#/definitions/Paging"

  Paging:
    properties:
      totalItems:
        type: integer
      totalPages:
        type: integer
      pageSize:
        type: integer
      currentPage:
        type: integer

# allOf 同样可以使用行内的数据定义
  PagedCollectingItems:
    allOf:
      - properties:
          items:
            type: array
            minItems: 10
            maxItems: 100
            uniqueItems: true
            items:
              $ref: "#/definitions/CollectingItem"
      - $ref: "#/definitions/Paging"
```

# 十一、Security 安全

安全策略类型： `basic`, `apiKey`, `oauth2`

```yaml
securityDefinitions: # 安全策略定义
  UserSecurity:
    type: basic
  AdminSecurity:
    type: basic
  MediaSecurity:
    type: basic
security: # 安全策略使用
  - UserSecurity: [] 
```

```yaml
# 基础鉴权
type: basic
```

```yaml
# API密钥鉴权
type: apiKey
name: api_key
in: header  # "query" or "header"
```

```yaml
# Oauth2鉴权
type: oauth2
authorizationUrl: http://swagger.io/api/oauth/dialog # oauth2 ("implicit", "accessCode")
tokenUrl: # oauth2 ("password", "application", "accessCode")
flow: implicit # "implicit", "password", "application" or "accessCode"
scopes:
  write:pets: modify pets in your account
  read:pets: read your pets
```

