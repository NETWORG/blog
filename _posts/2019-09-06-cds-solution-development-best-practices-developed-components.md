---
author: Jan Kostejn
title: CDS Solution Development – Best Practices – Developed Components
slug: cds-solution-development-best-practices-developed-components
id: 909
date: '2019-09-06 07:35:21'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Best Practices
  - Business Apps
  - BusinessApps
  - CDS
  - Common Data Service
  - Common Data Service for Apps
  - Microsoft
  - NETWORG
  - Power Platform
  - PowerApps
  - powerplatform
  - Solution Development
  - TALXIS
---

## Introduction.

Hi all! Another week went by, another [page in the GitHub wiki](https://github.com/NETWORG/cds-solution-development-docs/wiki/Developed-Components) added… In case you’re new here, read [this article]/2019/08/22/common-data-service-solution-development-best-practices/) which explains the motivation behind it all.

There's not a lot I can comment on for these few new pages. But I can think of one thing.

## What should be the output of my build?

I've seen many different CI/CD for CDS (Dynamics) and they were almost the same every time. It was focused on extensions, server-side and client-side. Usually the output of a build contained assemblies and compiled JavaScript code.

We are targeting CDS, we are targeting Power Platform. Power Platform is customized through '.zip' packages which are called solutions.

So the answer is simple - the output of your build should contain packed solutions. You can even go further and pack these solutions into package and deploy it through PackageDeployer. That's what every ISV should do.

## What's the output of your build?