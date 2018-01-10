# async-validator 中文文档

用作异步验证表单，源码地址：https://github.com/yiminghe/async-validator 。

以下**API**是对[async-validate](https://github.com/freeformsystems/async-validate)早期版本所做的修改。

## 1. 基本用法：

定义一个`描述器(descriptor)`作为`模式(schema)`实例，并使用`schema`的`validate`方法来验证。调用`validate`方法时传入参数：要被验证的`object`和`callback`回调函数。

```javascript
var schema = require('async-validator');
var descriptor = {
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({name: "muji"}, (errors, fields) => {
  if(errors) {
    // validation failed, errors is an array of all errors
    // fields is an object keyed by field name with an array of
    // errors per field
    return handleErrors(errors, fields);
  }
  // validation passed
});
```

## 2. Validate 验证方法：

```javascript
function(source, [options], callback)
```

* `source`: 被验证对象 (必须).
* `options`: 一个对象：验证过程选项 (可选).
* `callback`: 回调函数：验证完成时引用 (必须).

### 2.1 Options 选项

* `first`: Boolean，当第一条`验证规则rule`产生一个`error`时调用`回调函数callback`，其他验证规则将不再处理。这样对包含`多条异步`验证规则（比如需要查询数据库）时很有用。

* `firstFields`: Boolean|String[]，当为`指定字段`时：指定字段中第一验证规则产生error即调用callback回调函数，相同字段其他验证规则不再处理。当为`true`时：意味针对所有字段。

## 3. Rules 验证规则：

验证规则是用来处理验证的具体函数。

```javascript
function(rule, value, callback, source, options)
```

* `rule`: 通常指定一个`field`属性来对应要被规则验证的`字段`的`名字name`。
* `value`: `原始对象`的要被规则验证的`属性值`。
* `callback`: 一旦验证完成时调用的回调函数，期待被传入一个`Error`实例数组，以此表明验证失败。
* `source`: 被传入`validate`方法的`原始对象`。
* `options`: 附加选项。
* `options.messages`: 包含报错信息的对象，将被`defaultMessages默认报错信息`深度合并。

`options`可用传递到`validate`方法上，因此你可以借此在`validate`方法上引用一些临时数据。需要注意：`options`对象有一些保留属性名，如：`messages`、`exception` 和 `error`，如果在`options`对象中使用保留的属性名，他们将被覆盖。

```javascript
var schema = require('async-validator');
var descriptor = {
  name(rule, value, callback, source, options) {
    var errors = [];
    if(!/^[a-z0-9]+$/.test(value)) {
      errors.push(
        new Error(
          util.format("%s must be lowercase alphanumeric characters",
            rule.field)));
    }
    callback(errors);
  }
}
var validator = new schema(descriptor);
validator.validate({name: "Firstname"}, (errors, fields) => {
  if(errors) {
    return handleErrors(errors, fields);
  }
  // validation passed
});
```
针对单个字段上面使用多条验证规则通常是非常有用的，验证规则只需要是一个**对象数组**。比如：

```javascript
var descriptor = {
  email: [
    {type: "string", required: true, pattern: schema.pattern.email},
    {validator(rule, value, callback, source, options) {
      var errors = [];
      // test if email address already exists in a database
      // and add a validation error to the errors array if it does
      callback(errors);
    }}
  ]
}
```

### 3.1 Type 类型

表明使用的验证器的类型`type`：

* `string`: 必须是 `string` 类型. `默认类型`
* `number`: 必须是 `number` 类型.
* `boolean`: 必须是 `boolean` 类型.
* `method`: 必须是 `function` 类型.
* `regexp`: 必须是 `RegExp` 实例 或者一个`string`（字符串可以转换为一个正则表达式实例）。
* `integer`: 必须是 `number` 类型，并且是一个**整数**。
* `float`: 必须是 `number` 类型 并且是一个**浮点数**。
* `array`: 必须是 `array` 类型，（用`Array.isArray`方法来做类型判断的）
* `object`: 必须是 `object` 类型， 不能是`Array.isArray`.
* `enum`: 值必须在 `enum` 中存在。
* `date`: 值必须能转换为 `Date` 类型。
* `url`: 必须是 `url` 类型.
* `hex`: 必须是 `hex` 类型.
* `email`: 必须是 `email` 类型.

### 3.2 Required

`required`表明字段必须在要校验的原始对象中存在。

### 3.3 Pattern

`pattern`表明一个正则表达式，值要通过验证，必须要匹配这个正则表达式。

### 3.4 Range

使用`min`和`max`属性来定义范围。用来比较`string`和`array`类型的`length`， 或者用来比较`number`类型的数值大小。

### 3.5 Length

通过规定`len`属性来验证字段的精确长度。相当于明确指明：`string`和`array`类型的`length`，或者`number`类型的数值大小。
`len`属性比`min`和`max`有更高的优先级。

### 3.6 Enumerable

指明`enum`类型type后，使用`enum`属性列出字段可用的值列表。比如：

```javascript
var descriptor = {
  role: {type: "enum", enum: ['admin', 'user', 'guest']}
}
```

### 3.7 Whitespace

典型应用：必须字段中，如果只包含空白字符将被视为错误。在`string`类型规则中添加`whitespace`属性并设置为真将检查字符串是否全部是空白字符组成。

如果你希望清洗用户输入的值，而不只是空白字符检查，请看[transform](#transform)例子。

### 3.8 Deep Rules

如果你的验证深入到对象的属性，你只需要把`object`或`array`类型的验证中指定一个`fields`属性。

```javascript
var descriptor = {
  address: {
    type: "object", required: true,
    fields: {
      street: {type: "string", required: true},
      city: {type: "string", required: true},
      zip: {type: "string", required: true, len: 8, message: "invalid zip"}
    }
  },
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // errors for street, address.city, address.zip and address.name
});
```
注意，父规则上的`required`属性将完全验证原始对象上的字段是否没有声明，如果你在父规则上未规定`required`属性，嵌套的深度验证规则将不执行。[动手测试下吧……]

深度验证的嵌套规则create了一个schema，因此你也可以规定`options`传入`schema.validate()`方法。

```javascript
var descriptor = {
  address: {
    type: "object", required: true, options: {single: true, first: true},
    fields: {
      street: {type: "string", required: true},
      city: {type: "string", required: true},
      zip: {type: "string", required: true, len: 8, message: "invalid zip"}
    }
  },
  name: {type: "string", required: true}
}
var validator = new schema(descriptor);
validator.validate({ address: {} }, (errors, fields) => {
  // now only errors for street and name
});
```

父规则也会被验证，如果你设置的有的话：

```javascript
var descriptor = {
  roles: {
    type: "array", required: true, len: 3,
    fields: {
      0: {type: "string", required: true},
      1: {type: "string", required: true},
      2: {type: "string", required: true}
    }
  }
}
```

以上例子中，如果提供`{roles: ["admin", "user"]}`给原始对象，将有两条错误产生，一个是数组长度不匹配，一个是错过必须的数组索引2实体。


### 3.9 defaultField

`defaultField`属性在`array`或`object`类型的规则上使用，用来验证`array`或`object`**“容纳”**的所有的值/属性值。比如：

```javascript
var descriptor = {
  urls: {
    type: "array", required: true,
    defaultField: {type: "url"}
  }
}
```
注意： `defaultField`属性是`fields`属性的扩展，参考[deep rules](#deep-rules)。

### 3.10 Transform

有时你需要在校验前对值做改变，比如：强制转换或数据清洗等，你可以在验证规则中加入一个`transform`函数。这样既可以在校验开始前转变值，也可以在校验后重新指定值回到原始对象上。

```javascript
var schema = require('async-validator');
var sanitize = require('validator').sanitize;
var descriptor = {
  name: {
    type: "string",
    required: true, pattern: /^[a-z]+$/,
    transform(value) {
      return sanitize(value).trim();
    }
  }
}
var validator = new schema(descriptor);
var source = {name: " user  "};
validator.validate(source, (errors, fields) => {
  assert.equal(source.name, "user");
});
```

上面例子中，如果没有`transform`函数，将因为输入值的首尾包含有空白字符而校验失败；但如果有`transform`函数的话，校验方法将通过，同时字段值也将获得清洗。

## 4. Messages 报错信息：

由于你的应用上层依赖，你可能需要i18n支持或处理不同的验证报错信息。

最简单的办法就是在验证规则中指定一个`message`：

```javascript
{name:{type: "string", required: true, message: "Name is required"}}
```

`message`可以是任何类型，比如jsx格式。

```javascript
{name:{type: "string", required: true, message: <b>Name is required</b>}}
```

一个模式的验证规则内，你可能需要有多个语言包，在这种情况下，不推荐复制成多个模式规则（没有任何意义），你只需准备自己需要的语言包，提供对应message并将其分配给模式schema就行了：

```javascript
var schema = require('async-validator');
var cn = {
  required: '%s 必填',
};
var descriptor = {name:{type: "string", required: true}};
var validator = new schema(descriptor);
// deep merge with defaultMessages
validator.messages(cn);
...
```

如果你正在自定义验证函数，最好的办法是将多个message字符串赋值给一个`messages`对象，然后在验证函数中通过`options.messages` 属性使用它。

## 5. validator 验证器：

对规定字段你也可以自定义一个验证函数。

```js
const fields = {
  asyncField:{
    validator(rule,value,callback){
      ajax({
        url:'xx',
        value:value
      }).then(function(data){
        callback();
      },function(error){
        callback(new Error(error))
      });
    }
  },
 
  promiseField:{
      validator(rule, value){
        return ajax({
          url:'xx',
          value:value
        });
      }
    }
};
```
