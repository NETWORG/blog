---
author: Jan Skala
title: Logic Apps foreach and variables
slug: logic-apps-foreach-and-variables
id: 356
date: '2018-11-10 13:36:20'
categories:
  - Azure
  - English
---

Sometimes we need to work with a variable inside a loop section. Whether it's a precomputation or just a helper variable. Logic Apps allows us to do so. Yet the variable must be initialized on a global level (above all loops). ![](/uploads/2018/10/var.png)

### Here comes the problem:![](/uploads/2018/10/If.png)

By default, foreach runs in parallel, in 20 threads (instances). Now, because there is no such thing as mutex in Logic Apps, there is no way how to create a critical section. Critical section is a section only one thread at a time can enter. That results in dirty reads. We can solve this problem by running the loop synchronously. You can do that by editing settings of the foreach block. ![](/uploads/2018/10/Foreach.png) Now only one thread at a time will execute the foreach loop and no other thread will modify our variable while we work with it.