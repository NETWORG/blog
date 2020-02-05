---
author: Jan Kostejn
title: >-
  Merging Forms and Views – managed vs unmanaged (Microsoft Power Platform and
  CDS)
slug: merging-forms-and-views
id: 471
date: '2019-03-07 10:49:13'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - ALM
  - CDS
  - Common Data Service
  - Common Data Service for Apps
  - forms
  - managed
  - PowerApps
  - powerplatform
  - unmanaged
  - views
---

## Introduction.

If you are developing Model-driven app (business application) above the Microsoft Power Platform and you want to follow the best practices, you should deliver your complete solutions (zip packages) as managed solutions to all downstream environments.

Your goal should be to deploy your Model-driven app through AppSource. To achieve that, your app should be separated in smaller packages that can work on their own. For better understanding let’s say, that your app is separated in three main groups.

Model solutions which contain fields, that you are using or plan to use in your Model-driven app.

Feature solutions which contain extension code, that adds another functionality to Microsoft Power Platform (Common Data Service in our case).

Presentation solutions which contain app modules, app sitemap, forms and views – everything that user sees.

So, you’ll deploy model solutions with presentation solutions, but features can be independent blocks that can be added on demand. This is important for your ALM (application lifecycle management). Imagine that on the next release, you just update model and presentation solution – if there’s need - and add one new feature solution on top of it.

My point is that development, deployment and support of your app is much easier with Microsoft Power Platform.

I’d like to share with you some thoughts about views and forms that I had when I was developing our product. It’s how I think you should modify views and forms for better supportability in the future. If you disagree with me, let me know in the comments.

## How to work with forms?

Well I’m almost sure, that everyone who is reading this knows, how to customize form. But I want to write some ideas about how I done it and what I think is the “best practice”.

First, you should initialize forms in one of the solutions. Why? Imagine that you develop multiple applications that use the same entities and you want to display the information from both apps on one main form per entity. For example: imagine that you have Sales app and Marketing app. If you are at main account form in Sales app, you want to see basic information from Marketing app on it without switching apps.

So, I think that the best way to achieve this is to initialize empty forms in model solution (because you want to have fixed ids) and then add forms definition through presentation solutions for individual apps.

## Okay, I hear you, show me some samples.

I’ll show you how I work with account main form in terms of our product (custom Model-driven app). I have one solution for default entity metadata, forms and views. In this solution I introduced our custom empty form, because I need to reference it from presentation solutions later.

<figure class="wp-block-image is-resized">![](/uploads/2019/03/emptyAccountForm.png)</figure>

This is all it takes to initializes empty form.

As a next step I took this solution with empty form definition and imported it as managed solution on my development environment (if you have very deep knowledge of Common Data Service you can make the changes in customizations.xml of the presentation solution, but I wanted to make sure, that all changes I made were done properly).

I made new solution on my development environment (presentation solution) with only one component – this empty managed main account form. Then I made all the changes and got my final version of the form and exported this solution.

And this is what I wanted to share with you guys, because I couldn’t find how does customizations above managed form work and I wasn’t even sure, if it is even possible to append some form definition with another solution. Short answer is: it worked, and I like the way how it is designed.

In the unmanaged version of the solution are changes held as a complete form, so you can import this unmanaged version without the empty base form in the downstream environment and it will be created:

<figure class="wp-block-image is-resized">![](/uploads/2019/03/accountFormUnmanaged.png)</figure>

You can see that the formid element in the beginning of the form definition is the same as the one in empty form. That’s because I made all changes on the referenced form. But in this unmanaged version, it looks as complete definition that could be taken and imported in another environment. If I would take this definition and changed the id, environment would take it as a new account main form.

It gets more interesting in the managed version of the presentation solution!

<figure class="wp-block-image is-resized">![](/uploads/2019/03/accountFormManaged-Copy.png)</figure>

Now this is the same form that is in the previous picture, but you can see new attributes, what’s the most interesting is “solutionaction” and “ordinalvalue”. It’s just description of what is added/deleted/modified and in what order it should be rendered.

## And how is this relevant for me?

Imagine that you need to change one form based on set of applications. Not even you can, but you can have it in specific order… It’s completely modular.

## Deep dive for curious people.

I was wondering, if I have empty base form in one solution and definition in other solution, what happens if I’ll delete the presentation solution with form definition? I’d like if it would delete all and left just the empty form. And it works that way.

You need to know how CDS is layering solutions if you want to understand this. If you import managed solution, a downstream environment will make a layer for every component in the solution – so the empty managed form exists in its own managed layer. And with the import of the managed presentation solution (with form definition) another layer is created on top of it. CDS is rendering all managed layers from bottom to top and unmanaged layer as the last one in the result.

With the following removal of the presentation solution, the form definition managed layer gets deleted and only empty form managed layer stays in the downstream environment. If you would like to delete this managed layer, you need to delete the first solution, that introduced the empty form. If you apply this approach the downstream environment would be as clean as at the start of this blogpost.

## And what about views?

It works the same way with views. The only difference is that the merging between layers doesn’t work. The last one wins. For example, look at my empty view for account:

<figure class="wp-block-image is-resized">![](/uploads/2019/03/emptyAccountView.png)</figure>

And this is definition from presentation solution:

<figure class="wp-block-image is-resized">![](/uploads/2019/03/accountView.png)</figure>

If I would take this managed view and changed it in unmanaged layer, the unmanaged version of view will be displayed, because there’s no merging.

## Conclusion.

As I promised in my last blogpost, this is something more technical and it demands deeper knowledge of how CDS works.

I’ve recently spent a lot of time on technical design of our product and the biggest obstacle was how do the forms work? How will we display information from different apps? I investigated it and this is what I found out. Happy powerplatforming!