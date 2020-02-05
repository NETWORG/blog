---
author: matej.samler@thenetw.org
title: 'Solution Import Failure Message: Cannot add a Root Component 80'
slug: solution-import-failure-message-cannot-add-a-root-component-80
id: 928
date: '2019-09-16 13:22:17'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Dynamics
  - Power Platform
  - powerplatform
---

  
Have you tried to upload your own solution containing a model driven app into your environment, but for some unexpected reason, you got a very non descriptive error that the solution just cannot be uploaded, and only after a closer inspection, you discovered that the solution cannot add Root Component of type 80 because it is not in the target system?  

I know I did, and here is how I fixed the problem.

Let’s recap:  

We have a solution containing a model driven application with all it needs: Some dashboards, AppModuleSiteMaps, both managed and unamanged, AppModules themselves as well as solution.xml and customization.xml. When trying to upload the managed solution, we get the following error, even though nothing has changed since the last time I tried uploading it without a problem:

<figure class="wp-block-image">![](/uploads/2019/09/Annotation-2019-09-13-171250.png)</figure>

Surprisingly, the unmanaged version of the same solution still works and gets uploaded just fine. So what is different?  

The difference lies in SolutionPackager.exe, especially in the latest (as of the date of writing this post) version of the tool. When building the solution with this version, we may notice this message in the output:  

<figure class="wp-block-image">![](/uploads/2019/09/MicrosoftTeams-image-11-1024x162.png)</figure>

The missing dashboards are nothing new in the structure that we are using, but the first line is alarming. The AppModule is not defined?  

The reason we get this error is because only one AppModule.xml is no longer enough to build both managed and unmanaged solutions. We now need to have separate files for this, just like we need it for the Site Maps.  

This can be confirmed further by creating a new Model Driven App in our environment, exporting the solution and extracting it with the solution packager. The newest version will give us the following:  

<figure class="wp-block-image is-resized">![](/uploads/2019/09/h.png)</figure>

So to summarize:  

If you run into this problem, here are the steps to take:

1.  **Update your developer tools, especially the SolutionPackager from the [Nugget](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/download-tools-nuget)**
2.  **Add a managed version of your AppModule.xml to your solution**
3.  **Build managed and unmanaged versions of your solutions**