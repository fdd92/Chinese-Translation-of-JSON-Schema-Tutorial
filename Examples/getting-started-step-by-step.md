# 手把手入门教学

## 介绍

下面的例子不完全包括 JSON Schema 能提供的所有限定值，如果想了解更多请访问 <http://json-schema.org/specification.html>。

假设我们正在设计一个基于 JSON 的产品目录。这份目录中的每个产品包括如下属性：

- 标识符： `productId`
- 产品名： `productName`
- 售价： `price`
- 可选标签： `tags`



例如：

``` json
{
    "productId": 1,
    "productName": "A green door",
    "price": 12.50,
    "tags": [ "home", "green" ]
}
```

虽然很简单，但这个例子有些疑问：

- 什么是 `productId` ？
- `productName` 是必须的吗？

- `price` 的值可以是 0 吗？
- 所有的 `tags` 都是 string 类型的吗？

当你正在讨论数据格式，你想要有一份关于数据键的含义的元数据，包括这些键的有效数据。 JSON Schema 是一个提议的 IETF 标准，用来回答这些问题。



##Schema

开始设计 schema，我们从一份基本的 Schema 说起。

我们从称为*关键字* 的4个属性开始说起，它们表示为 [JSON](https://www.json.org/) 的键。

> 是的，这份标准使用了一个 JSON数据文档来描述数据。通常我们会使用 JSON 格式，但也可以用其他格式，比如 `text/xml`。

-  [`$schema`](http://json-schema.org/latest/json-schema-core.html#rfc.section.7) 关键词指的是这份 schema 是根据哪份指定的标准草案编写的，主要被用作版本控制。
- [`$id`](http://json-schema.org/latest/json-schema-core.html#rfc.section.8.2) 关键词定义 schema 的 URI。这个基础的 URI 用作这个 schema 里其他 URI 的引用。
- [`title`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.10.1) `和 `[`description`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.10.1) 注释关键词只用作注释用。它们不会对要验证的数据起到约束的作用，使用这两个关键字说明 schema 的意图。
- [`type`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.1.1) 验证关键词定义了我们 JSON 数据的第一个约束，它要求我们的数据必须是一个 JSON 对象。

``` json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product in the catalog",
    "type": "object"
}
```

当我们开始学习模式的时候引入如下术语：

- [Schema 关键词](http://json-schema.org/latest/json-schema-core.html#rfc.section.4.3.1)：`$schema` 和 `$id`
- [Schema 注释](http://json-schema.org/latest/json-schema-validation.html#rfc.section.10)：`title` 和 `description `
- [验证关键词](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6)： `type`



## 属性定义

``productId`` 是一个数值类型的产品唯一标识。它是产品规范标识，没有它的产品是不合理的，所以这个值是必须存在的。

所以我们在我们的schema 中加入新的JSON Schema术语：

- [`properties`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.4) 验证关键词
- `productId` 键
  - `description` schema 注释和 `type` 验证关键词 – 我们在上一部分介绍了这些。
- [`required`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.3) 验证关键词列表中添加 `productId`

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        }
    },
    "required": [ "productId" ]
}
```

- `productName` 是用来描述一个产品的字符串数值，因为没有名字的产品并不多，所以它也是必须的。
- 由于 `required` 校验关键词是数组类型，所以我们可以标记多个键为 “required”，我们现在将 `productName` 加入列表。
- `productId` 与 `productName` 之前没什么真正的区别  – 计算机通常更关注 ID 而人们更关注文字名称，所以我们保证这两者的完整性。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "productName": {
            "description": "Name of the product",
            "type": "string"
        }
    },
    "required": [ "productId", "productName" ]
}
```



## 深入了解属性

据店主说，他们没有免费的产品。;)

- `price` 键添加之前涵盖的常规 `description` schema 注释和 `type` 校验关键词。它同样包含在 `required` 校验关键词数组中。
- 我们指定 `price` 的值必须大于 0 使用 [`exclusiveMinimum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.5) 校验关键词。
  - 如果我们想包含 0 为合法数值，我们可以用 [`minimum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.4) 校验关键词。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "productName": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "description": "The price of the product",
            "type": "number",
            "exclusiveMinimum": 0
        }
    },
    "required": [ "productId", "productName", "price" ]
}
```

接下来，我们来看 `tags` 键。

店主是这么说的：

- 如果有 tags 键， 那必须包含至少一个标签。
- 所有的标签都需要相互独立；在同一个产品内没有重复标签。
- 所有的标签都需要是文本类型。
- 标签虽好，但不是必须存在的。

因此：

- `tags` 键添加了常规的注释和关键词。
- 这次 `type` 关键词是一个数组。
- 我们引入 [`items`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.1) 校验关键词以便我们可以定义数组中出现的内容。在这个例子里：`type` 关键词的值是 `string`。
- [`minItems`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.4) 校验关键词用于确保至少有个项目在数组中。
- [`uniqueItems`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.5) 关键词表明了数组中所有项目都必须是唯一的。
- 我们不需要添加这个键到 `required` 检验关键字数组里，因为它是可选的。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "productName": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "description": "The price of the product",
            "type": "number",
            "exclusiveMinimum": 0
        },
        "tags": {
            "description": "Tags for the product",
            "type": "array",
            "items": {
                "type": "string"
            },
            "minItems": 1,
            "uniqueItems": true
        }
    },
    "required": [ "productId", "productName", "price" ]
}
```



## 嵌套数据结构

到目前为止我们都在处理一个非常平面的 schema – 它只有一个层级。这部分我们展示一个嵌套的数据结构：

- 在这里添加使用我们先前提到的概念创建了 ``dimensions`` 键。由于`type` 校验关键词的值是 `object` ，所以我们可以使用 `properties` 校验关键词来定义一个嵌套数据。
  - 为简洁起见，我们省略了 `description` 注释关键词。但通常在这种情况下我们都应该完整的注释以便大多数开发者熟悉键和结构。
- 你会注意到 `required` 校验关键词仅应用于 dimensions 键而不是其他键。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "productName": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "description": "The price of the product",
            "type": "number",
            "exclusiveMinimum": 0
        },
        "tags": {
            "description": "Tags for the product",
            "type": "array",
            "items": {
                "type": "string"
            },
            "minItems": 1,
            "uniqueItems": true
        },
        "dimensions": {
            "type": "object",
            "properties": {
                "length": {
                    "type": "number"
                },
                "width": {
                    "type": "number"
                },
                "height": {
                    "type": "number"
                }
            },
            "required": [ "length", "width", "height" ]
        }
    },
    "required": [ "productId", "productName", "price" ]
}
```



## schema外部引用

到目前为止我们的 JSON schema 完全是自包含的。但为了重用、可读性和维护性等其他原因，在许多数据结构中共享 JSON schema 是非常常见的。

对于这个例子，我们引入一个带有两个属性的新的  JSON schema：

- 我们使用前面提到的 ``minimum`` 验证关键词。
- 我们添加 [`maximum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.2) 验证关键词。
- 综上，我们提供了一个用于校验的范围。

```json
{
    "$id": "https://example.com/geographical-location.schema.json",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "title": "Longitude and Latitude",
    "description": "A geographical coordinate on a planet (most commonly Earth).",
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

接下来我们引入这个 JSON schema。

```json
{
    "$schema": "http://json-schema.org/draft-07/schema#",
    "$id": "http://example.com/product.schema.json",
    "title": "Product",
    "description": "A product from Acme's catalog",
    "type": "object",
    "properties": {
        "productId": {
            "description": "The unique identifier for a product",
            "type": "integer"
        },
        "productName": {
            "description": "Name of the product",
            "type": "string"
        },
        "price": {
            "description": "The price of the product",
            "type": "number",
            "exclusiveMinimum": 0
        },
        "tags": {
            "description": "Tags for the product",
            "type": "array",
            "items": {
                "type": "string"
            },
            "minItems": 1,
            "uniqueItems": true
        },
        "dimensions": {
            "type": "object",
            "properties": {
                "length": {
                    "type": "number"
                },
                "width": {
                    "type": "number"
                },
                "height": {
                    "type": "number"
                }
            },
            "required": [ "length", "width", "height" ]
        },
        "warehouseLocation": {
            "description": "Coordinates of the warehouse where the product is located.",
            "$ref": "https://example.com/geographical-location.schema.json"
        }
    },
    "required": [ "productId", "productName", "price" ]
}
```

## 符合 JSON Schema 的数据示例

从我们最早的样本数据里拓展了很多概念（翻滚到最顶端查看）。让我们看下这份匹配我们定义的  JSON Schema 数据。

```json
{
    "productId": 1,
    "productName": "An ice sculpture",
    "price": 12.50,
    "tags": [ "cold", "ice" ],
    "dimensions": {
        "length": 7.0,
        "width": 12.0,
        "height": 9.5
    },
    "warehouseLocation": {
        "latitude": -78.75,
        "longitude": 20.4
    }
}
```

