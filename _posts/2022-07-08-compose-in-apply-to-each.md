---
author: Ondrej Juda
title: Remove variables from apply to each action
date: '2022-06-27T15:00:00+0200'
categories:
  - Power Platform
  - Power Automate
  - Optimization
tags:
  - Power Automate
  - Microsoft Flow
  - Optimization
---

Using variables inside your flows can be quite handy. You can store data in them, update the data, and append it to string or array variables. There is one slight problem with them. When you want to set or use a variable inside of an apply to each action, it comes with a cost. You can't use concurrency/parallelism.

### What is concurrency?

This out-of-box functionality can significantly speed up your flows. When you turn it on, you allow the flow to run multiple loops at the same time. Let's say you have listed 50 table rows and you want to do something with all the records. You can put it in the apply to each action and do the logic there.

![apply-to-each](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-01.png)

![settings](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-02.png)

The concurrency is turned off by default. So by default the apply to each action will process every row one by one - synchronously.
When you turn it on, you can set the value of concurrency from 1 to 50. When you set the concurrency to 50, you tell the flow it can process 50 loops at the same time. Since you listed 50 rows, it will be all handled at the same time.

### Why you shouldn't use concurrency with variables?

There is a slight problem when using variables. Let's say you update a variable and then use it a few actions later inside your apply to each loop. Problem is that the loop doesn't have its own context of the variable so if you set it and try to use it later, it could be already changed to some other value by some parallel loop. Even power automate does not suggest turning on concurrency when you use variables inside of it.

### You can solve it using compose action.

I would say that compose is my favorite action in power automate. I try to use it instead of variables whenever I can. I use variables only when there is no other way. Firstly let me introduce a flow using variables and I will show you, how to change it to compose.

I have here a flow, that retrieves national holidays of my country. Holidays returns as an array of objects. I need to create a new array, that contains only the dates of those holidays. How it would usually go, you would create an array variable, use apply to each method, and append the date to it. The flow would look like this:

![variables-flow](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-03.png)

Compose action has one special feature. When you use it inside of an apply to each, it turns into an array, when you refer to it outside of the apply to each action. Let me explain it in our example.

![compose-flow](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-04.png)

As you can see I removed the array variable and added compose action inside the apply to each instead. When I refer to outputs of the Compose-Date action, the outcome is the same as if I would use an array variable. I also turned on the concurrency and the apply for each action finished 4 times faster than with variable and without concurrency.

### It's too good. It must have some flaws.

If you think it can't be this perfect, then you are right. There are 3 little flaws I have to address.

#### Dynamic content

If you are using a variable, you will find it inside of the dynamic content selector. When you want to use this approach, you won't find the compose there. You have to use an expression to get the data outside. But don't be afraid, it's pretty simple. You just have to use expressions [outputs](https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference#outputs) and pass the name of the compose action as a parameter.

![outputs-expression](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-05.png)

#### Conditional compose

Let's make our example more complicated and add a condition. We want only holidays, that are not in April. 

![april](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-06.png)

If you compare our new outcome with the previous one, you can notice that there are two null values instead of the holidays in April. It is because the compose did not run when the condition outcome was false when the holidays were in April.
Same as in the first addressed issue, there is a simple trick to remove the null value - [filter](https://docs.microsoft.com/en-us/power-automate/data-operations#use-the-filter-array-action) action.
You just write a condition to filter out null values and the output of that filter action will be a new array without the null values.

![filter-edit](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-07.png)

![filter-outcome](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-08.png)

#### Nested apply to each action

If you create a more complex flow, where you need to use this approach inside of an apply for each action, so you would end up with nested loops (an apply for each inside of another apply for each), power automate won't let you save the flow. Fortunately there is a quick solution for this one. You just need to create a child flow, that will handle the nested apply for each and return the outcome you want.

### Should I stop using variables?

The answer is no. There are some situations, where the compose action won't be enough for your problem. The reason why I try to avoid using variables is performance. Variables slow your flows and it is not only when working with concurrency. You can read more about it in this article - [Variables or Compose?](https://sharepains.com/2021/01/06/variables-or-compose-power-automate/).

### What can I expect?
In the end, I would like to present the outcome of our optimization of one complex flow. We have a flow that is dealing with more than 1000 table rows. There were used a lot of variables inside of nested apply for each and do until actions to handle a calculation logic. The flow was so slow it took about 8 hours to finish.

![before-time](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-09.png)

![before-flow](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-10.png)

Thanks to removing the variables and turning on concurrency, a few handy [expressions](https://docs.microsoft.com/en-us/azure/logic-apps/workflow-definition-language-functions-reference), and a child flow we managed to make it quite a lot faster. From 8 hours just to 8 minutes.

![after-time](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-11.png)

![after-flow](/uploads/2022/07/2022-07-08-compose-in-apply-to-each-12.png)

I think you will agree that the flow looks much cleaner and the performance improvement is incredible. Hope that you will find this post useful and it will help you to make your flows faster.