---
author: Jan Kostejn
title: CDS Solution Development – Best Practices – ModelFeatureApp Architecture
slug: cds-solution-development-best-practices-modelfeatureapp-architecture
id: 938
date: '2019-09-19 12:16:49'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - Best Practices
  - CDS
  - Microsoft
  - NETWORG
  - PowerApps
  - powerplatform
  - Solution Development
  - TALXIS
---

## Introduction.

Hi all! Another week went by, another [page in the GitHub wiki](https://github.com/NETWORG/cds-solution-development-docs/wiki/Buildings) added… In case you’re new here, read [this article](https://blog.thenetw.org/2019/08/22/common-data-service-solution-development-best-practices/) which explains the motivation behind it all.

I wasn't able to add new content last week and I'm very sorry for that. Hopefully this week will make up for it.

## Buildings.

I've added two pages - one is for the [buildings](https://github.com/NETWORG/cds-solution-development-docs/wiki/Buildings) and one for the [building architecture](https://github.com/NETWORG/cds-solution-development-docs/wiki/Architecture). I believe that this is the most important thing if you want to make your app right.

Create architecture design and stick to it.

We've seen a lot of implementations and almost all of them were awful. Total ignorance of solution layering, unmanaged solutions on all downstream environments and cross dependencies between all solutions.

This is caused by no instructions on how to create solutions and what components should be included in one.

We invested a lot of time into designing architecture for our apps and I believe you can take a lot from it. Even the [Microsoft Solution Lifecycle Management whitepaper](https://www.microsoft.com/en-us/download/details.aspx?id=57777&WT.mc_id=rss_alldownloads_all) suggest something similar that we've came up with.

## How do you structure your solutions?