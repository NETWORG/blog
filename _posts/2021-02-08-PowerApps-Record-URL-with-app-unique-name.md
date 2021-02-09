---
author: Zdenek Srejber
title: >-
  PowerApps Record URL in combination with App Unique Name
date: '2021-02-08T15:00:00+0200'
categories:
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - Model-driven Apps
  - Record URL
  - App Unique Name
excerpt_separator: <!--more-->
---

In this blog post, we will showcase how to create a PowerApps Record URL which included App Unique Name instead of App ID.

# Why

The added benefit exists because App ID might change after import or deploy to a new environment, but App Unique Name is not likely to do so.

# Problem

 ## Old Way

What we did until now, is that we had a PowerAutomate flow where we used 'List Rows' action and app unique name as filter to find app ID.

![](/uploads/2021/02/2021-02-09-list-rows-app-id.png)

From there we were able to create a link in a way that would redirect you to a specific record in a specific app. You can see such URL below, there is an address followed by app id, entity name, and record id. This URL is exactly what you get if you want to share a record through email or use LevelUp For Dynamics to get Record URL.

```
https://yourenvurl.dynamics.com/main.aspx?appid=2b85482f-d660-eb11-89f5-000d3aa8e247&etn=networg_entityname&id={B63CD698-C066-EB11-A812-000D3A2EFE3A}&newWindow=true&pagetype=entityrecord
```

## Desired State

But, we aim to replace appid part for an app uniquename (see below), and retain the record id part. This way we can skip one action in Powerautomate flow.

```
https://yourenvurl.crm4.dynamics.com/apps/uniquename/app_uniquename
```

# Solution

We have struggled for a while to combine the two above mentioned URLs, yet the solution was quite simple. You will start with URL that includes uniquename, then apppend /main.aspx? and specify parameters of a record and window to open in.

```
https://yourenvurl.crm4.dynamics.com/apps/uniquename/['app_uniquename']/main.aspx?etn=['ent_logicalname']&id=['record_id']&newWindow=true&pagetype=entityrecord
```

['app_uniquename'] = unique name of your app

['ent_logicalname'] = logical name of record entity

['record_id'] = guid of a record