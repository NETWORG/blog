---
author: jan.kostejn@thenetw.org
title: Common Data Service Solution Development - Best Practices
slug: common-data-service-solution-development-best-practices
id: 874
date: '2019-08-22 07:57:28'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - ALM
  - BusinessApps
  - cdm
  - CDS
  - NETWORG
  - powerplatform
  - slm
  - Solution
  - solutions
  - TALXIS
---

## Introduction.

I started with series of articles about **managed and unmanaged** solutions from the ISV (Independent Software Vendor) perspective. You can find the first about merging forms and layering views [here](https://blog.thenetw.org/2019/03/07/merging-forms-and-views/), and the second about missing dependencies [here](https://blog.thenetw.org/2019/06/11/missing-dependencies-managed-vs-unmanaged-part-2-microsoft-power-platform-and-cds/). I was thinking about the next topic in the managed vs. unmanaged series and decided **to start over with best practices for CDS (Common Data Service) in general**.

I just want to state one thing. We are ISV. We **build and endorse managed solutions** and these best practices are our own based on the documentation from Microsoft and implementations of customer projects.

## Why?

As I said we are ISV and we are currently pushing our first product TALXIS Sales Start into the AppSource. We have this application completely in version control and **we learned a lot thanks to that**. We want to share this knowledge with the world, because not every partner is as experienced as us and we want the customer to be satisfied no matter how many ISVs are there. Since there's currently no complete **'how to' guide on CDS**, I feel that we have a lot to say!

Imagine situation where ISV called NETWORG has simple customer relationship management app and your company has field service solution. **The customer wants both**. There're two possibilities, you either fight for the customer and will have to create new CRM app, or **you cooperate with the other ISV** and are able to deliver instantly. The best thing about it is that the **customer is happy** - that's everyone's goal.

But when you meet with other ISVs on one customer you can't really **guarantee the quality** of solutions, that aren't yours... And that's the reason why we are making this documentation public. **We want to make sure, that every solution can meet our quality standards**.

## Where?

You can find the CDS best practices documentation **[here on this GitHub Wiki](https://github.com/networg/cds-solution-development-docs/wiki)**. I'll be adding pages usually once per week there. In addition, there's always going to be very **brief blogpost** about the new page here on the blog.

The [first page](https://github.com/NETWORG/cds-solution-development-docs/wiki/Engineer's-Guide-to-CDS) is just a small overview of solutions in general.

## Let's do the solution lifecycle management properly.