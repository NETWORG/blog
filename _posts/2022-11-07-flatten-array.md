---
author: Ondrej Juda
title: Flatten nested arrays
date: '2022-11-07T15:00:00+0200'
categories:
  - Power Platform
  - Power Automate
  - Arrays
tags:
  - Power Automate
  - Microsoft Flow
  - Arrays
---

I want to write a follow-up for one of my previous posts [Remove variables from apply to each action](/2022/06/27/compose-in-apply-to-each/). I have been using this technique quite a lot and run into a problematic situation with nested arrays.

Imagine you have a flow like this:

![flow](/uploads/2022/11/2022-11-07-flatten-array-01.png)

I list employees and I want to aggregate their open and closed opportunities. I want to use the compose action at the end of my apply for each action, so I can refer to it outside of the loop as I explained in the previous post.

![aggregation](/uploads/2022/11/2022-11-07-flatten-array-02.png)

Problem is that I need to have an array where the item will be an object, not another array. Usually, things don't go as we want and the outcome of this loop in the Compose-Data action looks like this, a nested array:

```json
[
  [
    {
      "name": "Jane Doe",
      "count": 3,
      "isClosed": true
    },
    {
      "name": "Jane Doe",
      "count": 8,
      "isClosed": false
    }
  ],
  [
    {
      "name": "Jonh Doe",
      "count": 3,
      "isClosed": true
    },
    {
      "name": "Jonh Doe",
      "count": 8,
      "isClosed": false
    }
  ]
]
```

I need it to look like this so I could work with it easier later:

```json
[
  {
    "name": "Jane Doe",
    "count": 3,
    "isClosed": true
  },
  {
    "name": "Jane Doe",
    "count": 8,
    "isClosed": false
  },
  {
    "name": "Jonh Doe",
    "count": 3,
    "isClosed": true
  },
  {
    "name": "Jonh Doe",
    "count": 8,
    "isClosed": false
  }
]
```

I came up with a solution, a primitive one, but it works for this scenario. We can create a string out of the array, and replace all square brackets with an empty string. This new string will be concatenated with a new pair of square brackets and converted back to json.

```
json(
    concat(
        '[',
        replace(
            replace(
                string(outputs('Aggregate-Opportunities')),
                '[',
                ''
            ),
            ']',
            ''
        ),
        ']'
    )
)
```

![correct outcome](/uploads/2022/11/2022-11-07-flatten-array-03.png)

While trying this approach, I run into one limitation. You can't use it when you have an array as a value inside an object.

```json
[
  {
    "A": [
      "B",
      "C"
    ]
  }
]
```

The expression above would fail to transform the string to json because it would look like this and it is an invalid json:
```json
[
  {
    "A": "B",
    "C"
  }
]
```