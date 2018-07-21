# mockjs

## 示例

```js
var Mock = require('mockjs')
// 模拟数据
var data = Mock.mock({
  'list|1-10': [{
    'id|+1': 1
  }]
})

console.log(JSON.stringify(data, null, 4))
```

```js
// 拦截ajax + 模拟数据
Mock.mock('http://123.com',{
  'name|3':'fei',
  'age|20-30':25,
})

$.ajax({
  url:'http://123.com',
  dataType:'json',
  success:function(e){
    console.log(e)
  }
})
```

## 语法

一、 **数据模板**中的每个属性由 3 部分构成： `'属性名|生成规则': 属性值`。

- `属性值`是是不同的数据类型时，`生成规则`代表的意义各不相同：
  - 属性值是`字符串`时： **重复**属性值的 次数 / 随机 **次数**范围;
    - `'name|count': value`
    - `'name|min-max': value`
  - 属性值是`数值`时：
    - 整数 / 浮点数整数部分 / 浮点数小数部分是 **固定值** / 随机 **大小**范围;
      - `'name|count': value`
      - `'name|min-max': value`
      - `'name|min-max.dmin-dmax': value`
      - `'name|min-max.dcount': value`
      - `'name|count.dmin-dmax': value`
      - `'name|count.dcount': value`
    - 以后的属性值 = 以属性值初始，**递加布数**;
      - `'name|+step': value`
  - 属性值是`布尔值`时： 属性值的 **概率**范围;
    - `'name|1': value` 1/2概率
    - `'name|min-max': value`
  - 属性值是 `对象`时： 抽取属性值的 **属性**的个数 / 随机 **个数**范围;
    - `'name|count': value`
    - `'name|min-max': value`
  - 属性值是 `数组`时： **抽取**属性值的 **一个元素** 或者 **重复**属性值的 **元素次数**、随机次数范围** ;
    - `'name|count': value`
    - `'name|min-max': value`
- 属性值还可以是`函数`或`正则表达式`……


二、 **占位符** 只是在`属性值`字符串中占个位置，并不出现在最终的属性值中。`@占位符(参数 [, 参数])`

- 数据类型： boolean, natural, integer, float, character, string, range, date, time, datetime, now
- 图片： image, dataImage
- 色彩： color
- 文字： paragraph, sentence, word, title, cparagraph, csentence, cword, ctitle
- 姓名： first, last, name, cfirst, clast, cname
- 网络： url, protocol, domain, email, ip, tld
- 地址： region, province, city, county?, zip
- 辅助： capitalize, upper, lower, pick, shuffle
- 其他： guid, id, increment
