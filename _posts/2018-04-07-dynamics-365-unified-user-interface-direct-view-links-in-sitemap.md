---
author: Tomas Prokop
title: 'Power Apps Sitemap: Open a specific view from sitemap'
slug: dynamics-365-unified-user-interface-direct-view-links-in-sitemap
id: 48
date: '2018-04-07 13:21:33'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Common Data Service for Apps
  - Dynamics 365
  - Model Driven App
  - SiteMap
  - Unified User Interface
---

You may need to point your sitemap items to a specific table view. You can achieve this by using a special URL in the sitemap sub area with the view's parameters and Power Apps host will manage to detect your intent when rendering the application navigation.

## Model-driven Power Apps (UCI) format:

<span style="color: #ff0000;">**/main.aspx?pagetype=entitylist&**</span>etn=**opportunity**&viewid={**83DCA8EC-B505-E811-80FB-00155D036800**}

## Old Dynamics 365 Web Client / MoCA format
If you used your existing sitemap from Dynamics 365 Web Client while creating an App or tried to use the former URL style, these custom links don't show up in the navigation panel of model-driven apps. Every App makes its own sitemap even when you use an existing solution to create the App so you don't need to worry about breaking your current user experience. 

<span style="color: #ff0000;">**/_root/homepage.aspx?**</span>etn=**opportunity**&viewid={**73E5C4A5-6727-E811-80FF-00155D036800**}

And now if you hit Publish button you will see respective sub areas in the navigation panel.

![](/uploads/2018/04/chrome_2018-04-07_13-46-58.png)

![](/uploads/2018/04/chrome_2018-04-07_13-39-43-300x260.png)

![](/uploads/2018/04/chrome_2018-04-07_13-49-55.png)

## Note

If your URL query contains entity type code that contains an integer code for the entity (e.g. etc=4) it is no longer supported. You should replace it with entity logical name of the entity (e.g. etn=account).
