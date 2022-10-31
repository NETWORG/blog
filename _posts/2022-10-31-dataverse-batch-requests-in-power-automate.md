---
author: Jan Kostejn
title: Dataverse Batch Requests in Power Automate
date: '2022-10-31T00:00:00+0200'
categories:
  - Power Platform
  - Power Automate
  - Custom Connectors
  - Dataverse
tags:
  - Power Platform
  - Power Automate
  - Microsoft Flow
  - Custom Connectors
  - Dataverse
---

## The Issue

As a part of the implementation, we were creating thousands of rows in Dataverse from external source. Let me start with the fact, that Power Automate is not the best choice when it comes to this scenario, because of [the limits](https://learn.microsoft.com/en-us/power-automate/limits-and-config#action-request-limits).

Proper solution would be to deploy a server-less code that would perform this integration task into the Azure. You are getting scalability and pay as you go model with that which is exactly the thing you are probably looking for.

We decided to use Power Automate in the end because of the unified deployment model - through managed solution - and because the fact, that everyone can open up and change a Power Automate flow since it's a tool with graphical and easy to use user interface. 

No matter the choice, you'll have to optimize your integration to avoid excessive api calls. [Read more about requests, limits and allocations in Power Platform](https://learn.microsoft.com/en-us/power-platform/admin/api-request-limits-allocations).

## The Solution

I performed a quick search and find out that every blogpost I saw is solving a batch operation in Power Automate through `Apply to Each` action. That means you do a separate API call for each record and you don't save your actions per flow either.

So how to achieve a proper Dataverse batch request in Power Automate? There are two main rules you must follow.

### Forbidden Apply to Each

You must avoid using [`Apply to Each`](https://learn.microsoft.com/en-us/power-automate/apply-to-each) action in your flow for transformation. If you'd use this action, you'll iterate through every record there is. Sure, that works for a hundred of records. But imagine having a collection of 70k records. Are you able to determine when your flow is going to finish? How many actions will be evaluated? How many requests you'll send?

Majority of your transformations can be solved by [`Filter Array`](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-filter-array-action) and [`Select`](https://learn.microsoft.com/en-us/power-automate/data-operations#use-the-select-action) actions instead of `Apply to Each`. 

If you require more complex transformation, I advise you to use some template language for this. In our use case, we created custom connector that can resolve [Liquid template language](https://shopify.github.io/liquid/).

### The Batch Request

[The Dataverse connector](https://learn.microsoft.com/en-us/connectors/commondataserviceforapps/) doesn't support a batch request. There's an option to [perform a changeset request](https://learn.microsoft.com/en-us/connectors/commondataserviceforapps/#perform-a-changeset-request), but that's something slightly different, it won't save you any API calls. That's where [the HTTP with Azure AD connector](https://learn.microsoft.com/en-us/connectors/webcontents/) comes in play. This one is extremely useful, because it lets you call any service of your choice as long as you're in your Azure Directory. This applies to Dataverse as well.

You won't be able to build your batch request body without the `Apply to Each` action, if you can't use external service (already mentioned liquid custom connector in our specific scenario).

### Demo

Bear with me...

![Dataverse Batch Request in Power Automate](/uploads/2022/10/batch-request-power-automate.png)

Let me start from the top. You can see `Liquid-Batch` action which is using the custom connector I've mentioned already. This connector has two inputs: _map_ - liquid transformation and _entity_ - data to be used for the transformation. We're using it here to build the body of the batch request.

#### Red Rectangle

This is where the magic happens, the iteration was moved from Power Automate to a tool that was designed for such thing - liquid. This is the reason, why we can avoid `Apply to Each` action and evaluate tens of thousands of records in seconds.

#### Green Rectangle

This is the mapping. If the property is evaluated as a null, it's inputted as a null value.

#### Orange Rectangle

These are the transformed data we want to import. I also added a property with the `batchId` generated in the first compose action. 

#### Blue Rectangle

The final step. Sending the batch request against our Dataverse environment. Body is output of the `Liquid-Batch` action - body of our request as a string.

## TL;DR

Just because you can do it, it doesn't mean you have to. If you must use batch requests, then your task probably requires something built up for it...

I encourage you to look into [dataflows](https://learn.microsoft.com/en-us/power-query/dataflows/overview-dataflows-across-power-platform-dynamics-365), because that's another option if you're not into writing code.