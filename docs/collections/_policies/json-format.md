---
layout: default
title: JSON policy format
nav_order: 7
---
<!-- markdownlint-disable-file MD036 -->

# JSON policy format {#json-format}
{: .no_toc }

To allow for programmatically constructing and parsing policies, Cedar supports a [JSON](https://json.org) policy format. This topic describes the JSON format for individual policies/templates and policy sets.

<details open markdown="block">
  <summary>
    Topics on this page
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Representing a policy with JSON

### Overview

A "standard" Cedar policy looks like the following:

```cedar
permit (
    principal == User::"12UA45",
    action == Action::"view",
    resource in Folder::"abc"
) when {
    context.tls_version == "1.3"
};
```

When you retrieve the JSON representation of this policy, it looks like the following:

```json
{
    "effect": "permit",
    "principal": {
        "op": "==",
        "entity": { "type": "User", "id": "12UA45" }
    },
    "action": {
        "op": "==",
        "entity": { "type": "Action", "id": "view" }
    },
    "resource": {
        "op": "in",
        "entity": { "type": "Folder", "id": "abc" }
    },
    "conditions": [
        {
            "kind": "when",
            "body": {
                "==": {
                    "left": {
                        ".": {
                            "left": {
                                "Var": "context"
                            },
                            "attr": "tls_version"
                        }
                    },
                    "right": {
                        "Value": "1.3"
                    }
                }
            }
        }
    ]
}

```

{: .note }
>In this topic, the term policy refers to both static policies and policy templates.

{: .note }
>The JSON representation of a Cedar policy does not preserve comments, whitespace, or newline characters.

The JSON representation of a policy contains the following keys:

* [effect](#effect)
* [principal](#principal)
* [action](#action)
* [resource](#resource)
* [conditions](#conditions)
* [annotations](#annotations)

### `effect`

The `effect` object is required.

The value of this object must be either the string `permit` or the string `forbid`.

```json
"effect": "permit",
"effect": "forbid",
```

### `principal`

The `principal` object is required.

The value of this object must include an object with the key `op`, and depending on the value of `op`, an object with the key `entity` or `slot`.

#### `op`

The `op` key is required. The `op` object must have one of the following string values:

* `All`

    If present, then the original policy contains only the keyword `principal` with no constraints. In this case, the `principal` object doesn't require any additional objects.

    **Example**

    Cedar policy line:

    ```cedar
    principal
    ```

    JSON representation:

    ```json
    "principal": {
        "op": "All"
    }
    ```

* `==`
    If present, then the `principal` object must also have one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    principal == User::"12UA45"
    ```

    JSON representation:

    ```json
    "principal": {
        "op": "==",
        "entity": { "type": "User", "id": "12UA45" }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    principal == ?principal
    ```

    JSON representation:

    ```json
    "principal": {
        "op": "==",
        "slot": "?principal"
    },
    ```

* `in`

  If present, then the `principal` object must also have one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    principal in Group::"Admins"
    ```

    JSON representation:

    ```json
    "principal": {
        "op": "in",
        "entity": { "type": "Group", "id": "Admins" }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    principal in ?principal
    ```

    JSON representation

    ```json
    "principal": {
        "op": "in",
        "slot": "?principal"
    },
    ```

* `is`

  If present, then the `principal` object must also have an `entity_type` key.

  **Example**

  Cedar policy line:

  ```cedar
  principal is User
  ```

  JSON representation:

  ```json
  "principal": {
      "op": "is",
      "entity_type": "User"
  }
  ```

  The `principal` object may also optionally have an `in` key. The value of this key is an object with one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    principal is User in Group::"Admins"
    ```

    JSON representation:

    ```json
    "principal": {
        "op": "is",
        "entity_type": "User",
        "in": {
            "entity": { "type": "Group", "id": "Admins" }
        }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    principal is User in ?principal
    ```

    JSON representation

    ```json
    "principal": {
        "op": "is",
        "entity_type": "User",
        "in": {
            "slot": "?principal"
        }
    },
    ```

### `action`

The `action` object is required.

The value of this object must include an object with the key `op`, and depending on the value of `op`, an object with the key `[entity](#entity)` or `[entities](#entities)`.

#### `op`

The `op` key is required.

The `op` object must have one of the following string values:

* `All`

    If present, then the original policy contains only the keyword `action` with no constraints. In this case, the `action` object doesn't require any additional objects.

    **Example**

    Cedar policy line:

    ```cedar
    action
    ```

    JSON representation:

    ```json
    "action": {
        "op": "All"
    }
    ```

* `==`

    If present, then the `action` object must also have the following object:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    action == Action::"readFile"
    ```

    JSON representation:

    ```json
    "action": {
        "op": "==",
        "entity": { "type": "Action", "id": "readFile" }
    }
    ```

* `in`

  If present, then the `action` object must also have one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    action in Action::"readOnly"
    ```

    JSON representation:

    ```json
    "action": {
        "op": "in",
        "entity": { "type": "Action", "id": "readOnly" }
    }
    ```

  * [`entities`](#entities)

    **Example**

    Cedar policy line:

    ```cedar
    action in [ Action:: "ManageFiles", Action::"readFile", Action::"writeFile", Action::"deleteFile"]
    ```

    JSON representation

    ```json
    "action": {
        "op": "in",
        "entities": [
            { "type": "Action", "id": "ManageFiles" }, // Action group
            { "type": "Action", "id": "readFile" },
            { "type": "Action", "id": "writeFile" },
            { "type": "Action", "id": "deleteFile" }
        ]
    }
    ```

### `resource`

The `resource` object is required.

The value of this object must include an object with the key `op`, and depending on the value of `op`, an object with the key `entity` or `slot`.

#### `op`

The `op` key is required.

The `op` object must have one of the following string values:

* `All`

    If present, then the original policy contains only the keyword `resource` with no constraints. In this case, the `resource` object doesn't require any additional objects.

    **Example**

    Cedar policy line:

    ```cedar
    resource
    ```

    JSON representation:

    ```json
    "resource": {
        "op": "All"
    }
    ```

* `==`

  If this operator is present, then the `resource` object must also have one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    resource == file::"vacationphoto.jpg"
    ```

    JSON representation:

    ```json
    "resource": {
        "op": "==",
        "entity": { "type": "file", "id": "vacationphoto.jpg" }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    resource == ?resource
    ```

    JSON representation:

    ```json
    "resource": {
        "op": "==",
        "slot": "?resource"
    },
    ```

* `in`

  If present, then the `resource` object must also have one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    resource in folder::"Public"
    ```

    JSON representation:

    ```json
    "resource": {
        "op": "in",
        "entity": { "type": "folder", "id": "Public" }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    resource in ?resource
    ```

    JSON representation

    ```json
    "resource": {
        "op": "in",
        "slot": "?resource"
    }
    ```

* `is`

  If present, then the `resource` object must also have an `entity_type` key.

  **Example**

  Cedar policy line:

  ```cedar
  resource is file
  ```

  JSON representation:

  ```json
  "resource": {
      "op": "is",
      "entity_type": "file"
  }
  ```

  The `resource` object may also optionally have an `in` key. The value of this key is an object with one of the following:

  * [`entity`](#entity)

    **Example**

    Cedar policy line:

    ```cedar
    resource is file in folder::"Public"
    ```

    JSON representation:

    ```json
    "resource": {
        "op": "is",
        "entity_type": "file",
        "in": {
            "entity": { "type": "Folder", "id": "Public" }
        }
    }
    ```

  * [`slot`](#slot)

    **Example**

    Cedar policy line:

    ```cedar
    resource is file in ?resource
    ```

    JSON representation

    ```json
    "resource": {
        "op": "is",
        "entity_type": "file",
        "in": {
            "slot": "?resource"
        }
    },
    ```

### conditions

The `conditions` object is required.

The value of this object is a JSON array of objects.  Each object in the array must have exactly two keys: `kind` and `body`.

The `kind` key must be either the string `when` or the string `unless`.

The `body` key must be an [JsonExpr object](#JsonExpr-objects).

**Example**

Cedar policy lines

```cedar
when { ... }
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            ...
        }
    }
]
```

### `annotations`

Annotations, if present, must be a JSON object.  The keys must be strings and the values may be strings or the JSON value `null`.
A `null` value corresponds to an annotation without a value (e.g., `@shadow_mode`) and is equivalent to the value `""`.

### JsonExpr objects {#JsonExpr-objects}

An JsonExpr object is an object with a single key that is any of the following.

+ [`Value`](#JsonExpr-Value)
+ [`Var`](#JsonExpr-Var)
+ [`Slot`](#JsonExpr-Slot)
+ [`Unknown`](#JsonExpr-Unknown)
+ [`!`, `neg`, and `isEmpty` operators](#JsonExpr-neg)
+ [Binary operators: `==`, `!=`, `in`, `<`, `<=`, `>`, `>=`, `&&`, `||`, `+`, `-`, `*`, `contains`, `containsAll`, `containsAny`, `hasTag`, `getTag`](#JsonExpr-binary)
+ [`.`, `has`](#JsonExpr-has)
+ [`is`](#JsonExpr-is)
+ [`like`](#JsonExpr-like)
+ [`if-then-else`](#JsonExpr-if-then-else)
+ [`Set`](#JsonExpr-Set)
+ [`Record`](#JsonExpr-Record)
+ [`Any other key`](#JsonExpr-any-other-key)

#### `Value` {#JsonExpr-Value}

The value of this key is a Cedar value in the same syntax as expected for entity attribute values in Cedar’s entity format. This can include entity reference literals, set literals, and record literals.

**Example with numeric literals**

Cedar policy line:

```cedar
when { 1 == 2 };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "==": {
                "left": {
                    "Value": 1
                },
                "right": {
                    "Value": 2
                }
            }
        }
    }
]
```

**Example with entity literals**

Cedar policy line

```cedar
when { User::"alice" == Namespace::Type::"SomePrincipal" };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "==": {
                "left": {
                    "Value": {
                        "__entity": {
                            "type": "User",
                            "id": "alice"
                        }
                    }
                },
                "right": {
                    "Value": {
                        "__entity": {
                            "type": "Namespace::Type",
                            "id": "SomePrincipal"
                        }
                    }
                }
            }
        }
    }
]
```

**Example with set literals**

Cedar policy line:

```cedar
when { [1, 2, "something"] == [4, 5, "otherthing"] };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "==": {
                "left": {
                    "Set": [
                        { "Value": 1 },
                        { "Value": 2 },
                        { "Value": "something" },
                    ]
                },
                "right": {
                    "Set": [
                        { "Value": 4 },
                        { "Value": 5 },
                        { "Value": "otherthing" },
                    ]
                }
            }
        }
    }
]
```

**Example with record literals**

Cedar policy line:

```cedar
when { {something: "spam", otherthing: false} == {} };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "==": {
                "left": {
                    "Record": {
                        "something": { "Value": "spam" },
                        "otherthing": { "Value": false },
                    }
                },
                "right": {
                    "Record": {}
                }
            }
        }
    }
]
```

#### `Var` {#JsonExpr-Var}

The value of this key is one of the strings `principal`, `action`, `resource`, or `context`.

**Example**

Cedar policy line:

```cedar
when { principal == action && resource == context };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "&&": {
                "left": {
                    "==": {
                        "left": {
                            "Var": "principal"
                        },
                        "right": {
                            "Var": "action"
                        }
                    }
                },
                "right": {
                    "==": {
                        "left": {
                            "Var": "resource"
                        },
                        "right": {
                            "Var": "context"
                        }
                    }
                }
            }
        }
    }
]
```

#### `Slot` {#JsonExpr-Slot}

The value of this key is one of the strings `?principal` or `?resource` and act as placeholders in [policy templates](templates.html). Currently, policies containing this are not valid Cedar.

#### `Unknown` {#JsonExpr-Unknown}

The value of this key is an object with a single key name, whose value is the name of the unknown. This is used for partial-evaluation.  In particular, these values may appear in the JSON rendering of residuals.

#### `!`, `neg`, and `isEmpty` operators {#JsonExpr-neg}

The value of this key is an object with a single key argument, whose value is itself an [JsonExpr object](#JsonExpr-objects).

**Example using `.` and `!`**

Example Cedar policy line:

```cedar
when { !context.something };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "!": {
                "arg": {
                    ".": {
                        "left": {
                            "Var": "context"
                        },
                        "attr": "something"
                    }
                }
            }
        }
    }
]
```

#### Binary operators: `==`, `!=`, `in`, `<`, `<=`, `>`, `>=`, `&&`, `||`, `+`, `-`, `*`, `contains`, `containsAll`, `containsAny`, `hasTag`, `getTag` {#JsonExpr-binary}

The value for any of these keys is an object with keys `left` and `right`, which are each themselves an [JsonExpr object](#JsonExpr-objects).

**Example for `contains`**

Cedar policy line

```cedar
when { principal.owners.contains("something") };
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "contains": {
                "left": {
                    ".": {
                        "left": {
                            "Var": "principal"
                        },
                        "attr": "owners"
                    }
                },
                "right": {
                    "Value": "something"
                }
            }
        }
    }
]
```

#### `.`, `has` {#JsonExpr-has}

The value of one of these keys is an object with keys `left` and `attr`.  The left key is itself an [JsonExpr object](#JsonExpr-objects), while the `attr` key is a string.

**Example for `.`**

Cedar policy line

```cedar
context.something
```

JSON representation

```json
".": {
    "left": {
        "Var": "context"
    },
    "attr": "something"
}
```

#### `is` {#JsonExpr-is}

The value of this key is an object with the keys `left` and `entity_type`.
The `left` key is itself an [JsonExpr object](#JsonExpr-objects), while the `entity_type` key is a string.
The value may optionally have an `in` key which is also a JsonExpr object.

**Example for `is`**

Cedar policy line

```cedar
principal is User in Group::"friends"
```

JSON representation

```json
"is": {
    "left": { "Var": "principal" },
    "entity_type": "User",
    "in": {"Value": {"__entity": { "type": "Folder", "id": "Public" }}}
}
```

#### `like` {#JsonExpr-like}

The value of this key is an object with keys `left` and `pattern`.
The left key is itself an [JsonExpr object](#JsonExpr-objects), while the `pattern` key is an array composed of pattern elements.
A pattern element can be one of either
  - the string `Wildcard`
  - an object with a single key `Literal`, whose value is a string

**Example for `pattern`**

The pattern `/home/alice/docs/*.txt` is represented as:

```json
"pattern": [
  {
    "Literal": "/home/alice/docs/"
  },
  "Wildcard",
  {
    "Literal": ".txt"
  }
]
```

Note that it's allowed to represent literals with up to one `Literal` per character.
For instance, in the previous example, we could use up to four objects with `Literal` keys to represent the `.txt` string.

#### `if-then-else` {#JsonExpr-if-then-else}

The value of this key is an object with keys `if`, `then`, and `else`, each of which are themselves an [JsonExpr object](#JsonExpr-objects).

**Example for if-then-else**

Cedar policy line

```cedar
when {
    if context.something
    then principal has "-78/%$!"
    else resource.email like "*@amazon.com"
};
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "if-then-else": {
                "if": {
                    ".": {
                        "left": {
                            "Var": "context"
                        },
                        "attr": "something"
                    }
                },
                "then": {
                    "has": {
                        "left": {
                            "Var": "principal"
                        },
                        "attr": "-78/%$!"
                    }
                },
                "else": {
                    "like": {
                        "left": {
                            ".": {
                                "left": {
                                    "Var": "resource"
                                },
                                "attr": "email"
                            }
                        },
                        "pattern": "*@amazon.com"
                    }
                }
            }
        }
    }
]
```

#### `Set` {#JsonExpr-Set}

The value of this key is a JSON array of values, each of which is itself an [JsonExpr object](#JsonExpr-objects).

**Example**

Cedar policy element

```cedar
[1, 2, "something"]
```

JSON representation

```json
{
    "Set": [
        { "Value": 1 },
        { "Value": 2 },
        { "Value": "something" },
    ]
}
```

#### `Record` {#JsonExpr-Record}

The value of this key is a JSON object whose keys are arbitrary strings and values are themselves [JsonExpr objects](#JsonExpr-objects).

**Example for record**

Cedar policy element
`{something: "spam", somethingelse: false}`

JSON representation

```json
{
    "Record": {
        "foo": { "Value": "spam" },
        "somethingelse": { "Value": false },
    }
}
```

#### Any other key {#JsonExpr-any-other-key}

This key is treated as the name of an extension function or method.  The value must be a JSON array of values, each of which is itself an [JsonExpr object](#JsonExpr-objects).  Note that for method calls, the method receiver is the first argument.  For example, for `a.isInRange(b)`, the first argument is for `a` and the second argument is for `b`.

**Example for `decimal` function**

Cedar policy line

```cedar
decimal("10.0")
```

JSON representation

```json
{
    "decimal": [
        {
            "Value": "10.0"
        }
    ]
}
```

**Example for `ip` function and `isInRange` method**

Cedar policy line

```cedar
when {
    context.source_ip.isInRange(ip("222.222.222.0/24"))
};
```

JSON representation

```json
"conditions": [
    {
        "kind": "when",
        "body": {
            "isInRange": [
                {
                    ".": {
                        "left": {
                            "Var": "context"
                        },
                        "attr": "source_ip"
                    }
                },
                {
                    "ip": [
                        {
                            "Value": "222.222.222.0/24"
                        }
                    ]
                }
            ]
        }
    }
]
```

## Representing a policy set with JSON {#policy-set-format}

### Overview

Here is an example policy set containing a static policy and policy template.

```cedar
permit (
    principal == User::"12UA45",
    action == Action::"view",
    resource in Folder::"abc"
);

forbid (
    principal == User::"12UA45",
    action == Action::"view",
    resource in ?resource
);
```

Here is the JSON representation of this policy set, plus a template-linked policy that sets the `?resource` placeholder of the template.

```json
{
    "staticPolicies": {
        "policy0": {
            "effect": "permit",
            "principal": {
                "op": "==",
                "entity": { "type": "User", "id": "12UA45" }
            },
            "action": {
                "op": "==",
                "entity": { "type": "Action", "id": "view" }
            },
            "resource": {
                "op": "in",
                "entity": { "type": "Folder", "id": "abc" }
            },
            "conditions": []
        }
    },
    "templates": {
        "template0": {
            "effect": "forbid",
            "principal": {
                "op": "==",
                "entity": { "type": "User", "id": "12UA45" }
            },
            "action": {
                "op": "==",
                "entity": { "type": "Action", "id": "view" }
            },
            "resource": {
                "op": "in",
                "slot": "?resource"
            },
            "conditions": []
        }
    },
    "templateLinks": [
        {
            "templateId": "template0",
            "newId": "policy1",
            "values": {
                "?resource": {
                    "type": "Folder",
                    "id": "def"
                }
            }
        }
    ]
}
```

The JSON representation of a policy set contains the following keys:

* [staticPolicies](#staticpolicies)
* [templates](#templates)
* [templateLinks](#templatelinks)

### `staticPolicies`

This field is the set of static policies in the policy set, represented as a map from policy ID to policy content in the format described [above](#representing-a-policy-with-json). This field can only include static policies; including a template will result in an error.

### `templates`

This field is the set of templates in the policy set, represented as a map from policy ID to template in the format described [above](#representing-a-policy-with-json).

### `templateLinks`

This field is a JSON array of template links. The JSON representation of a template link contains the following keys:

* `templateId`
* `newId`
* `values`

`templateId` is the ID of the policy to be linked against. `new_id` is the ID of the newly generated template-linked policy, and `values` is a mapping from slots (`?principal` or `?resource`) to entities.

## Convert Cedar policy format to JSON policy format using Java

**Note**: These steps are specific to Java, the process will be different using other languages.

If you have a policy statement in Cedar policy format, for example if you've used Amazon Verified Permission's [BatchGetPolicy](https://docs.aws.amazon.com/verifiedpermissions/latest/apireference/API_BatchGetPolicy.html) API operation, and want to convert it to JSON policy format, follow these steps:

1. Get the policy statement by calling the [parseStaticPolicy](https://github.com/cedar-policy/cedar-java/blob/main/CedarJava/src/main/java/com/cedarpolicy/model/policy/Policy.java) method.
2. Convert the policy statement to JSON by calling the [toJson](https://github.com/cedar-policy/cedar-java/blob/main/CedarJava/src/main/java/com/cedarpolicy/model/policy/Policy.java) method.
3. Convert the string representation of the policy statement to JSON policy format using [gson](https://github.com/google/gson) or a similar tool.

Now you can easily parse the principal, action, resource, and conditions included in the policy.
