---
author: Tomas Prokop
title: 'Quick Fix: Microsoft Flow Error - Invalid type. Expected String but got Null.'
slug: quick-fix-microsoft-flow-error-invalid-type-expected-string-but-got-null
id: 498
date: '2019-03-11 08:30:23'
categories:
  - Microsoft Flow
tags:
  - Error
  - Microsoft Flow
  - powerplatform
  - Troubleshooting
---

Hi, today I came across JSON parser error in Microsoft Flow. I used auto-generated schema and everything had been working just fine until a connector I used had started to return null values for strings.

<div class="wp-block-image">

![](/uploads/2019/03/chrome_2019-03-11_08-30-13-1024x654.png)

</div>

The fix is quite easy. You just need to manually modify a type of property in the schema. By default you get type **String** but it is not nullable type.

Fortunatelly JSON Parser in Microsoft Flow can handle multiple types in the schema:

```json
    {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "Description": {
            "type": [
              "string",
              "null"
            ]
          }
        }
      }
    }
```

So you just need to change

    "type": "string"

to

    "type": ["string", "null"]

![](/uploads/2019/03/chrome_2019-03-11_08-39-41-1024x699.png)