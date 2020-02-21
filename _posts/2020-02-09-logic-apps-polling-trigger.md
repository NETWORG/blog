---
author: Jan Skala
title: >-
  Support Logic Apps Polling Trigger in your .NET Core Web API
slug: >-
  logic-apps-polling-trigger
date: '2020-02-29 09:00:00'
categories:
  - Azure
  - Azure Logic Apps
  - Power Platform
tags:
  - Azure
  - Azure Logic Apps
  - Power Platform
---

Are you familiar with Logic Apps? Have you ever written your custom connector and wondered how to support Polling Triggers in your API? The way is not exactly straightforward, until recently. In [Networg](https://networg.com/) we created many custom connectors with polling triggers. We assembled an easy to use tooling for that and finally made it open source. Now you can easily add Polling Trigger to your .NET Core API using our [NuGet package](https://www.nuget.org/packages/FluentPollingTriggerBuilder/1.0.0). If you are interested in contribution or just want to see our source code, you can check out our [GitHub repo](https://github.com/NETWORG/Utilities.LogicApps.FluentPollingTriggerBuilder).

If you are confused about all this, it's ok. The documentation of Logic App triggers is everything but ideal. Out entire progress with this problem was made mainly by reverse engineering. Here is a simple presentation, which explains how it works and what exactly does this NuGet for you.

<iframe src="//www.slideshare.net/slideshow/embed_code/key/1JWWGctE8FlrGW" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>
