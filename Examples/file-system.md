# 使用 JSON Schema 为文件系统建模

## 介绍

> 并非仅使用 JSON Schema 就可以对所有的 fstab 文件约束进行建模；然而它可以代表大多数的情况，并且这个练习对演示约束如何工作非常有用。提供的示例主要为了说明JSON Schema的概念而不是fstab文件系统的实际工作模式。

这个例子现实文件系统挂载点可能的 JSON Schema 的表示，如 [`/etc/fstab`](https://en.wikipedia.org/wiki/Fstab) 所示。

fstab 文件中的条目可以有非常多种形式；这里是一个例子：

```json
{
    "/": {
        "storage": {
            "type": "disk",
            "device": "/dev/sda1"
        },
        "fstype": "btrfs",
        "readonly": true
    },
    "/var": {
        "storage": {
            "type": "disk",
            "label": "8f3ba6f4-5c70-46ec-83af-0d5434953e5f"
        },
        "fstype": "ext4",
        "options": [ "nosuid" ]
    },
    "/tmp": {
        "storage": {
            "type": "tmpfs",
            "sizeInMB": 64
        }
    },
    "/var/www": {
        "storage": {
            "type": "nfs",
            "server": "my.nfs.server",
            "remotePath": "/exports/mypath"
        }
    }
}
```



## 创建 `fstab文件` schema

我们将从表示如下约束的基础 JSON Schema 开始：

- 这个条目列表试一个 JSON 对象。
- 这个对象的成员名（或者属性名）必须全部是有效的绝对路径；
- 必须要有一个条目表示文件系统根路径（如，`/`）

从上到下搭建我们的 JSON Schema：

- [`$id`](http://json-schema.org/latest/json-schema-core.html#rfc.section.8.2) 关键字
- [`$schema`](http://json-schema.org/latest/json-schema-core.html#rfc.section.7) 关键字
- [`type`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.1.1) 校验关键字
- [`required`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.3) 校验关键字
- [`properties`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.4) 校验关键字
  - 现在 `/` 键是空的，我们稍后补充。
- [`patternProperties`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.5) 校验关键字
  - 通过正则表达式匹配其他属性名。注意：它不匹配 `/`。
  - `^(/[^/]+)+$` 键现在是空的，我们稍后补充。
- [`additionalProperties`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.5.6) 校验关键字
  - 此处值是 `false` ，表示约束对象属性名是 `/` 或匹配正则表达式。

> 你会注意到这里的正则表达式是显式锚定的（使用 `^` 和 `$`）：在 JSON Schema 中默认不会锚定正则表达式（在`patternProperties` 和 `pattern` 中）。（译注：锚定似乎是指正则匹配使用 `^` 和 `$` 来匹配完整的字符串，而 JSON Schema 允许只匹配部分）

```json
{
    "$id": "http://example.com/fstab",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "required": [ "/" ],
    "properties": {
        "/": {}
    },
    "patternProperties": {
        "^(/[^/]+)+$": {}
    },
    "additionalProperties": false,
}
```



## `条目` schema

我们从添加了我们已经讨论过的新概念的 JSON Schema 的大纲说起。

我们在之前的练习中已经看过了这些关键词：`$id`, `$schema`, `type`, `required` 和`properties`。

为此我们添加：

- [`description`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.10.1) 注释关键词。
- [`oneOf`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.7.3) 关键词。
- [`$ref`](http://json-schema.org/latest/json-schema-core.html#rfc.section.8.3) 关键词。
  - 在这个例子里，所有的引用都是使用了关系路径（`#/...`）的本地引用。
- [`definitions`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.9) 关键词。
  - 包含了我们稍后将会定义的几个键。

```json
{
    "$id": "http://example.com/entry-schema",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "description": "JSON Schema for an fstab entry",
    "type": "object",
    "required": [ "storage" ],
    "properties": {
        "storage": {
            "type": "object",
            "oneOf": [
                { "$ref": "#/definitions/diskDevice" },
                { "$ref": "#/definitions/diskUUID" },
                { "$ref": "#/definitions/nfs" },
                { "$ref": "#/definitions/tmpfs" }
            ]
        }
    },
    "definitions": {
        "diskDevice": {},
        "diskUUID": {},
        "nfs": {},
        "tmpfs": {}
    }
}
```



## 约束一个条目

我们现在来扩展这个骨架，为某些属性添加约束。

- 我们的 `fstype` 键使用 [`enum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.1.2) 校验关键字。
- 我们的 `options` 键使用如下内容：
  - `type` 校验关键字（见上文）。
  - [`minItems`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.4) 校验关键字。
  - [`items`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.1) 校验关键字。
  - [`uniqueItems`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.4.5) 校验关键字。
  - 总的来说：`options` 必须是一个数组，项目必须是字符串类型，必须包含一个项目，并且所有项目都应该是唯一的。
- 我们有一个 `readonly` 键。

在我们添加了这些约束后，这个 schema 现在变成了这个样子：

```json
{
    "$id": "http://example.com/entry-schema",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "description": "JSON Schema for an fstab entry",
    "type": "object",
    "required": [ "storage" ],
    "properties": {
        "storage": {
            "type": "object",
            "oneOf": [
                { "$ref": "#/definitions/diskDevice" },
                { "$ref": "#/definitions/diskUUID" },
                { "$ref": "#/definitions/nfs" },
                { "$ref": "#/definitions/tmpfs" }
            ]
        },
        "fstype": {
            "enum": [ "ext3", "ext4", "btrfs" ]
        },
        "options": {
            "type": "array",
            "minItems": 1,
            "items": {
                "type": "string"
            },
            "uniqueItems": true
        },
        "readonly": {
            "type": "boolean"
        }
    },
    "definitions": {
        "diskDevice": {},
        "diskUUID": {},
        "nfs": {},
        "tmpfs": {}
    }
}
```



## `diskDevice` 定义

引入一个一个新的关键词：

- [`pattern`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.3.3) 验证关键词表示 `device` 键必须是以 */dev* 开头的绝对路径

```json
{
  "diskDevice": {
    "properties": {
      "type": {
        "enum": [ "disk" ]
      },
      "device": {
        "type": "string",
        "pattern": "^/dev/[^/]+(/[^/]+)*$"
      }
    },
    "required": [ "type", "device" ],
    "additionalProperties": false
  }
}
```



## `diskUUID` 定义

这里没有介绍新的关键字。

我们有一个新的键：`label`，并且用 `pattern` 校验关键词确保它是一个合法的 UUID。

```json
{
    "diskUUID": {
        "properties": {
            "type": {
                "enum": [ "disk" ]
            },
            "label": {
                "type": "string",
                "pattern": "^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12}$"
            }
        },
        "required": [ "type", "label" ],
        "additionalProperties": false
    }
}
```



## `nfs` 定义

我们发现了另外的新关键字：

- [`format`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.7) 注释和断言关键字。

```json
{
    "nfs": {
        "properties": {
            "type": { "enum": [ "nfs" ] },
            "remotePath": {
                "type": "string",
                "pattern": "^(/[^/]+)+$"
            },
            "server": {
                "type": "string",
                "oneOf": [
                    { "format": "hostname" },
                    { "format": "ipv4" },
                    { "format": "ipv6" }
                ]
            }
        },
        "required": [ "type", "server", "remotePath" ],
        "additionalProperties": false
    }
}
```



## `tmpfs` 定义

我们最后一个定义介绍了两个新的关键字：

- [`minimum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.4) 校验关键字。

- [`maximum`](http://json-schema.org/latest/json-schema-validation.html#rfc.section.6.2.2) 校验关键字。

- 他们一起要求大小必须在 16 到512 之间（包括16和512）。

```
{
  "tmpfs": {
    "properties": {
      "type": { "enum": [ "tmpfs" ] },
      "sizeInMB": {
        "type": "integer",
        "minimum": 16,
        "maximum": 512
      }
    },
    "required": [ "type", "sizeInMB" ],
    "additionalProperties": false
  }
}
```



## 完整的条目schema

最终的 schema 非常的大：

```json
{
    "$id": "http://example.com/entry-schema",
    "$schema": "http://json-schema.org/draft-07/schema#",
    "description": "JSON Schema for an fstab entry",
    "type": "object",
    "required": [ "storage" ],
    "properties": {
        "storage": {
            "type": "object",
            "oneOf": [
                { "$ref": "#/definitions/diskDevice" },
                { "$ref": "#/definitions/diskUUID" },
                { "$ref": "#/definitions/nfs" },
                { "$ref": "#/definitions/tmpfs" }
            ]
        },
        "fstype": {
            "enum": [ "ext3", "ext4", "btrfs" ]
        },
        "options": {
            "type": "array",
            "minItems": 1,
            "items": {
                "type": "string"
            },
            "uniqueItems": true
        },
        "readonly": {
            "type": "boolean"
        }
    },
    "definitions": {
        "diskDevice": {
            "properties": {
                "type": {
                    "enum": [ "disk" ]
                },
                "device": {
                    "type": "string",
                    "pattern": "^/dev/[^/]+(/[^/]+)*$"
                }
            },
            "required": [ "type", "device" ],
            "additionalProperties": false
        },
        "diskUUID": {
            "properties": {
                "type": {
                    "enum": [ "disk" ]
                },
                "label": {
                    "type": "string",
                    "pattern": "^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12}$"
                }
            },
            "required": [ "type", "label" ],
            "additionalProperties": false
        },
        "nfs": {
            "properties": {
                "type": { "enum": [ "nfs" ] },
                "remotePath": {
                    "type": "string",
                    "pattern": "^(/[^/]+)+$"
                },
                "server": {
                    "type": "string",
                    "oneOf": [
                        { "format": "hostname" },
                        { "format": "ipv4" },
                        { "format": "ipv6" }
                    ]
                }
            },
            "required": [ "type", "server", "remotePath" ],
            "additionalProperties": false
        },
        "tmpfs": {
            "properties": {
                "type": { "enum": [ "tmpfs" ] },
                "sizeInMB": {
                    "type": "integer",
                    "minimum": 16,
                    "maximum": 512
                }
            },
            "required": [ "type", "sizeInMB" ],
            "additionalProperties": false
        }
    }
}
```



## 在 `fstab` schema 中引用`条目` schema

最后，我们用 `$ref` 关键字添加我们的`条目` schema 到键左边的空值里：

- `/` 键
- `^(/[^/]+)+$` 键

```json
{
  "$id": "http://example.com/fstab",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": [ "/" ],
  "properties": {
    "/": { "$ref": "http://example.com/entry-schema" }
  },
  "patternProperties": {
    "^(/[^/]+)+$":  { "$ref": "http://example.com/entry-schema" }
  },
  "additionalProperties": false,
}
```

