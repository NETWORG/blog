---
author: Ondrej Juda
title: There was a problem refreshing the dataflow
date: '2022-08-22T11:00:00+0200'
categories:
  - Power Platform
  - Power App
  - Power Query Dataflows
  - 
tags:
  - Power Apps
  - Power Query
  - Dataflows
  - Error
---

Lately, I got my hands on [Power Apps Power Query Dataflows](https://docs.microsoft.com/en-us/power-apps/maker/data-platform/self-service-data-prep-with-dataflows){:target="_blank"}. It is a pretty handy tool for migrating, transforming, and importing data. I suggest you to try it for yourself. Even though it is a helpful tool, I must admit that I didn't have a good experience with it when trying to put it through [Application Lifecycle Management](https://docs.microsoft.com/en-us/power-platform/alm/overview-alm){:target="_blank"} and deploy it using Azure Pipelines. But this is not the topic of this article. I want to show you an error I ran into.

![Error](/uploads/2022/08/2022-08-22-dataflow-unspecified-error-01.png)

As you can see, the error tells you nothing about what is happening. It happened when I finished data transformation, mapped columns to my table in Dataverse, and tried to import the data. Since I didn't know what it was about, I had to remove data transformation steps one by one to find out what was wrong. Fortunately, it didn't take long.

The problem was caused by the mapping of [alternate key](https://docs.microsoft.com/en-us/power-apps/developer/data-platform/define-alternate-keys-entity){:target="_blank"} on lookup. These keys are essential, If you want to map a lookup. It is used instead of the unique id of a row. Without it, the lookup won't even show on the mapping page. In my example, I was importing expenses from a CSV file. On the expense in Dataverse, I wanted to hold information about measurement units - liters, kilograms, pieces,... At the moment, when I populated the key for the unit and refreshed the dataflows, the error appeared.

![Mapping](/uploads/2022/08/2022-08-22-dataflow-unspecified-error-02.png)

I went to check back on the alternate key for the Measurement Unit table and I found, that it had some status and it said **Failed**. When you create an alternate key, it needs to go through some initialization process before you can use it properly and for some reason, this one failed.

![Alternate key failed](/uploads/2022/08/2022-08-22-dataflow-unspecified-error-03.png)

To cut it short, the problem was in data. I wanted to use a measurement unit symbol as an alternate key for mapping, but we had duplicate data in it. What I didn't think about was that two units could have the same symbol.

![Duplicate symbol](/uploads/2022/08/2022-08-22-dataflow-unspecified-error-04.png)

My solution was to change the alternate key from symbol to name, the error disappeared and the dataflow started to work.

![Import succeeded](/uploads/2022/08/2022-08-22-dataflow-unspecified-error-05.png)