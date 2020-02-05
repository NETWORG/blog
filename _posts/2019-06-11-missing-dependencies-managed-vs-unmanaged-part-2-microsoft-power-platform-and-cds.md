---
author: Jan Kostejn
title: >-
  Missing Dependencies - managed vs unmanaged PART 2 (Microsoft Power Platform
  and CDS)
slug: >-
  missing-dependencies-managed-vs-unmanaged-part-2-microsoft-power-platform-and-cds
id: 654
date: '2019-06-11 16:30:57'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - active
  - CDS
  - Common Data Service
  - Common Data Service for Apps
  - DevOps
  - managed
  - missing dependencies
  - solutions
  - unmanaged
---

## Introduction.

This is the second part in my series of CDS/Dynamics solution development.

[In the last article]/2019/03/07/merging-forms-and-views/), I described why **you should use managed solutions** and why it's not a great idea to use unmanaged ones. I want to support my suggested best practices in this series in the future so you have the option to make your own opinion. I'm strongly for managed solutions and I'll tell you another reason why in this part.

Have you ever heard of **"missing dependencies"**? Well if you are CDS developer like me, I know you have and I know that they are pain in your ***.

Actually we have [an article here in this blog]/2018/06/24/remove-missing-dependencies-from-solution-xml-with-powershell/), which provides you with tooling to automatically remove missing dependencies, in case that you don't have the luxury of source control and aren't able to deploy due to **false** missing dependencies.

Well how does it happen that **false missing dependency is created** and you can't deploy your solution into downstream environment? Why do you have to delete the element manually or use script for this? **Why does it even exist in the first place?!**

Yeah, I know, I was asking myself the same questions and I'll provide you with answers today.

## What is "missing dependency"?

First of all we should understand why there is such a thing. If you already have overview of the _solution.xml_ file, you can skip this part.

[](/uploads/2019/06/1.png)

It's really simple as you can see. Every missing dependency contains a component **(and managed solution in which the component exists)** that is required by different dependent component.

## Why missing dependencies exist?

This is something that came with the solution framework and it makes perfect sense. Imagine situation where you have a lot of solutions and you aren't able to keep track of **what has to be imported before a random solution**. You could try, wait until the end of an import dialog and then the import would fail because of one or more missing components. Or you could find out right away - missing dependencies, **they are checked even before the import starts**.

## It sounds like a good thing, why do people struggle with them?

Missing dependencies work perfectly if you are developing solutions the right way. Wait, there's nothing like the right way... Let's say the better way. What I mean is that the development needs to be independent of the environment. If you want to use missing dependencies the way it was intended, you can have only **one unmanaged solution per environment** (I know that this sounds really extreme, but it's Microsoft's recommendation based on the latest [Solution Lifecycle Management Document - CDS ALM](https://www.microsoft.com/en-us/download/details.aspx?id=57777)), because when you have all of the other solutions managed, you can't create dependency to the "**Active**" (unmanaged) layer. If that can't happen, there should be no **false** missing dependencies and all should work as intended.

## How can I achieve this?

The easiest way to achieve this is to **move your customizations to the version control**. For example, we are developing solutions independently and committing into the Azure DevOps. This has one downside to it - there are no <MissingDependencies /> from the start and during the development, but in the end, there is a simple way how to get this element filled properly.

You can get it filled by importing unmanaged solution to an environment, where every other solution is managed (remember how we talked about one unmanaged solution per environment - this is it). If you do this, **CDS will automatically update <MisingDependencies /> element** and you can export the solution back to your computer and obtain the missing dependencies.

I said that moving customizations into the Azure DevOps had **one downside** for us. At least that's how I presented it... **But look at it from the other angle:** It enables us to keep one unmanaged solution per environment, because we have every solution in both forms **built on demand**. So our "downside" is easily solvable and we can create quality managed solutions for our customers.