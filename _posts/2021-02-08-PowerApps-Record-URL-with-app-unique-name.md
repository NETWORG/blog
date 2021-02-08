---
author: Zdeněk Šrejber
title: >-
  PowerApps Record URL in combination with App Unique Name
date: '2021-02-08T15:00:00+0200'
categories:
  - URL
tags:
  - Record URL
  - App Unique Name
  - Dev
excerpt_separator: <!--more-->
---

In this blogpost we will showcase how to create PowerApps Record URL which included App Unique Name instead of App ID.

This has added benefit, becouse App ID might change after import or deploy to new environment, but App Unique Name is not likely to do so.

What we did until now, is that we used a PowerAutomate flow where we used app unique name just to find app ID, from there we were able to create a link in a way that would redirect you to specific record in a specific app. You can see such URL below, there is an address followed by app id, entity name and record id. This URL is exactly what you get if you want to share a record through email or use LevelUp For Dynamics to get Record URL.

https://yourenvurl.dynamics.com/main.aspx?appid=2b85482f-d660-eb11-89f5-000d3aa8e247&etn=networg_entityname&id={B63CD698-C066-EB11-A812-000D3A2EFE3A}&newWindow=true&pagetype=entityrecord

But, our aim is to replace appid part for app uniquename (see below), and retain record id part. This way we can skip one action in Powerautomate flow.

https://yourenvurl.crm4.dynamics.com/apps/uniquename/networg_uniqueappname

We have struggled for a while to combine the two above mentioned URLs, yet the solution was quite simple. You will start with url that includes unieqname then inset /main.aspx? and specified parameters of a record and window to open in.

https://yourenvurl.crm4.dynamics.com/apps/uniquename/networg_uniqueappname/main.aspx?etn=networg_entityname&id=B63CD698-C066-EB11-A812-000D3A2EFE3A&newWindow=true&pagetype=entityrecord
