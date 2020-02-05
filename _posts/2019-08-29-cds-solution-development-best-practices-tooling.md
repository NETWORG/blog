---
author: Jan Kostejn
title: CDS Solution Development - Best Practices - Tooling
slug: cds-solution-development-best-practices-tooling
id: 894
date: '2019-08-29 15:50:12'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Best Practices
  - CDS
  - Common Data Service
  - Common Data Service for Apps
  - Microsoft
  - NETWORG
  - PCF
  - Power Platform
  - PowerApps
  - PowerApps Component Framework
  - powerplatform
  - Solution Development
  - TALXIS
---

## Introduction.

Hi all! Another week went by, another [page in the GitHub wiki](https://github.com/NETWORG/cds-solution-development-docs/wiki/Tooling) added... In case you're new here, read [this article](/2019/08/22/common-data-service-solution-development-best-practices/) which explains the motivation behind it all.

What are my thoughts on current CDS tooling? Let's get into it.

## What to use?

I think that the worst thing about CDS tooling is that there's so many options. What to choose? My recommendation is pretty straightforward - stick to first party (Microsoft's) tools as much as possible. That are the tools that Microsoft uses in their development and supports them.

## Are Microsoft tools good enough?

Well in reality a lot of the tools from MS are just not enough. There's a lot of them and they sometimes doesn't work as they should. For example there's warning in SolutionPackager.exe if you try to pack form as a root component, because the tool thinks that it's dashboard. That's happening as long as I remember.

What I want to say is we stick to first party tools whenever possible... But in some situations where the tooling doesn't work, you need to find workaround and solve the task on your own.

In our case, that was building PCF controls. It's currently in preview and it's changing a lot. We can't be dependent on something so volatile, so we did create our own build process that's using scripts from original PCF tooling, but it's triggered differently.

There's much more and I'm sure that you're able to imagine what I'm talking about.

## What's next for CDS tooling?

I complaint a little, but I'm very happy with the direction things are taking. PCF tooling is great example. Maybe it's not perfect, but it's still under development and I'm really looking forward the 1.0.0 version. Another great step is the [PowerApps BuildTools for Azure DevOps](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/build-tools-overview), which finally gives us option to work with environments from DevOps. Again, it's not perfect - I'm not sure if they support deployments through certificate (must have for us), but we finally see more and more guidance from Microsoft.

## What tooling do you use?