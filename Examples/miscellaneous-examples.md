# 更多例子



## 基础

这个例子提供了你可能在  JSON Schema 中看过的典型最小值限制。它包括：

- [`$id`](http://json-schema.org/latest/json-schema-core.html#rfc.section.8.2) 关键字
- [`$schema`](http://json-schema.org/latest/json-schema-core.html#rfc.section.7) 关键字
- [`title`](http://json-schema.org/latest/json-schema-hypermedia.html#rfc.section.6.5.1) 注释关键字
- [`type`](http://json-schema.org/latest/json-schema-core.html#rfc.section.4.2.1) 实例数据模型
- [`properties`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.4) 校验关键字
- 三个键：`firstName`，`lastName` 和 `age` ，他们每个都拥有：
  - [`description`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.10.1) 注释关键字。
  - `type` 实例数据模型（见上文）。
- [`minimum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.4) 校验关键字，用于 `age` 键。

```json
{
    "$id": "https://example.com/person.schema.json",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Person",
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "description": "The person's first name."
        },
        "lastName": {
            "type": "string",
            "description": "The person's last name."
        },
        "age": {
            "description": "Age in years which must be equal to or greater than zero.",
            "type": "integer",
            "minimum": 0
        }
    }
}
```

**数据**

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "age": 21
}
```



## 地理坐标描述

这个例子介绍了：

- [`required`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.3) 校验关键字
- [`minimum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.4) 校验关键字
- [`maximum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.2) 校验关键字

```json
{
    "$id": "https://example.com/geographical-location.schema.json",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Longitude and Latitude Values",
    "description": "A geographical coordinate.",
    "required": [ "latitude", "longitude" ],
    "type": "object",
    "properties": {
        "latitude": {
            "type": "number",
            "minimum": -90,
            "maximum": 90
        },
        "longitude": {
            "type": "number",
            "minimum": -180,
            "maximum": 180
        }
    }
}
```

**数据**

```json
{
    "latitude": 48.858093,
    "longitude": 2.294694
}
```



## 数组之类的东西

数组在 JSON 里面是基础的结构 – 这里我们演示了几种可以描述的方法：

- 字符串数组。
- 对象数组。

我们还通过这个例子介绍了如下内容：

- [`definitions`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.9) 关键字
- [`$ref`](http://json-schema.org/latest/json-schema-core.html#rfc.section.8.3) 关键字

```json
{
    "$id": "https://example.com/arrays.schema.json",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "description": "A representation of a person, company, organization, or place",
    "type": "object",
    "properties": {
        "fruits": {
            "type": "array",
            "items": {
                "type": "string"
            }
        },
        "vegetables": {
            "type": "array",
            "items": { "$ref": "#/definitions/veggie" }
        }
    },
    "definitions": {
        "veggie": {
            "type": "object",
            "required": [ "veggieName", "veggieLike" ],
            "properties": {
                "veggieName": {
                    "type": "string",
                    "description": "The name of the vegetable."
                },
                "veggieLike": {
                    "type": "boolean",
                    "description": "Do I like this vegetable?"
                }
            }
        }
    }
}
```

**数据**

```json
{
  "fruits": [ "apple", "orange", "pear" ],
  "vegetables": [
    {
      "veggieName": "potato",
      "veggieLike": true
    },
    {
      "veggieName": "broccoli",
      "veggieLike": false
    }
  ]
}
```

