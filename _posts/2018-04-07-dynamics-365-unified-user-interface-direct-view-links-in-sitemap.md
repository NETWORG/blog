---
author: Tomas Prokop
title: 'Dynamics 365 Unified User Interface: Direct View Links in SiteMap'
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

You may have used direct links to specific view in the past but you don't see these sub areas now in your UUI App. You could have achieved this by using a special URL in the sitemap sub area with the view's parameters and Dynamics managed to detect your intent when rendering the application navigation. This is still working but the format has changed for UUI Apps.

## Web Client / MoCA

![](/uploads/2018/04/chrome_2018-04-07_13-39-43-300x260.png) To achieve this you can go to your SiteMap editor and create an URL Sub Area with the following link: <span style="color: #000000;">/_root/homepage.aspx?e</span>tn=**opportunity**&viewid={**73E5C4A5-6727-E811-80FF-00155D036800**} ![](/uploads/2018/04/chrome_2018-04-07_13-49-55.png)

## Unified User Interface

If you have used your existing sitemap while creating an App or tried to use the former approach, these custom links don't show up in the navigation panel. Every App makes its own sitemap even when you use an existing solution to create the App so you don't need to worry about breaking your current user experience. To make it work you just need to make a slight modification in the URL.

#### New format:

<span style="color: #ff0000;">**/main.aspx?pagetype=entitylist&**</span>etn=**opportunity**&viewid={**83DCA8EC-B505-E811-80FB-00155D036800**}

#### Former:

<span style="color: #ff0000;">**/_root/homepage.aspx?**</span>etn=**opportunity**&viewid={**73E5C4A5-6727-E811-80FF-00155D036800**} And now if you hit Publish button you will see respective sub areas in the navigation panel. ![](/uploads/2018/04/chrome_2018-04-07_13-46-58.png)

## Note

If your URL query contains entity type code that contains an integer code for the entity (e.g. etc=4) it is no longer supported. You should replace it with entity logical name of the entity (e.g. etn=account).