---
author: Jan Skala
title: Internal NuGet feed with Azure DevOps (VSTS)
slug: internal-nuget-feed-with-azure-devops-vsts
id: 509
date: '2019-03-26 14:01:44'
categories:
  - Azure
  - English
  - Uncategorized
tags:
  - Azure DevOps
  - Dev
  - NuGet
---

Imagine you have some tools or a framework you want to share with your company and reuse it on various projects. If you develop your tool in .Net standard, then you are in luck. Creating a NuGet package has never been easier.

## Make your library a NuGet

For the sake of this demo, I will make a simple project.

    public static class Calculator
        {
            public static int Plus(int a, int b)
            {
                return a + b;
            }
        }

Now we have to configure this project, so we get a .nupkg file as a build output.

### Option 1: Project Properties - package

<figure class="wp-block-image">![](/uploads/2019/03/image-1024x596.png)</figure>

Make sure to check **Generate NuGet package on build**. Then you can fill whatever info you need for your package.

### Option 2: Edit csproj

<figure class="wp-block-image">![](/uploads/2019/03/image-1.png)</figure>

Now when we build our solution or the project we receive the .nupkg as an output.

    skala@DESKTOP-J3SNLJP /m/c/U/j/s/r/C/Calculator
    dotnet build
    Microsoft (R) Build Engine version 15.9.20+g88f5fadfbe for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

      Restore completed in 66.99 ms for /mnt/c/Users/jskal/source/repos/Calculator/Calculator/Calculator.csproj.
      Calculator -- /mnt/c/Users/jskal/source/repos/Calculator/Calculator/bin/Debug/netstandard2.0/Calculator.dll
      Successfully created package '/mnt/c/Users/jskal/source/repos/Calculator/Calculator/bin/Debug/Calculator.1.0.0.nupkg'.

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:02.33
    skala@DESKTOP-J3SNLJP /m/c/U/j/s/r/C/Calculator
    ls bin/Debug/
    Calculator.1.0.0.nupkg*  netstandard2.0/

## Setup build and release pipeline

I assume you have some basic experience with Azure DevOps (VSTS). If not, here is a quick [how to](https://docs.microsoft.com/en-us/azure/devops/user-guide/).

### Build pipeline

1.  Use Nuget 4.3.0
2.  Nuget restore
3.  Build solution (Visual studio build)
4.  Copy files to $(build.artifactstagingdirectory)
    *   By default, .nupkg(s) are not part of the publish artifacts
    *   Contents = **/*.nupkg
    *   Target folder = $(build.artifactstagingdirectory)
5.  Publish Artifact: drop

<figure class="wp-block-image">![](/uploads/2019/03/j9TDhGpNrS.gif)

<figcaption>Step by step build pipeline creation</figcaption>

</figure>

### NuGet feed

Before we create our release pipeline, we need to create a Nuget feed. Navigate to Artifacts and click on "New feed" button. Give it a name and you are ready to go.

<figure class="wp-block-image">![](/uploads/2019/03/hnnSuUdZg8.gif)

<figcaption>Step by step NuGet feed creation</figcaption>

</figure>

### Release pipeline

Our pipeline will simply deliver all .nupkg(s) to our NuGet feed which we created in previous step.

<figure class="wp-block-image">![](/uploads/2019/03/releasepipeline.gif)

<figcaption>Step by step release pipeline creation</figcaption>

</figure>

Now your release pipeline will automatically trigger every time your build pipeline is successful. And this is how the output looks like.

<figure class="wp-block-image">![](/uploads/2019/03/image-3-1024x517.png)</figure>

### Side note

I had to change my **PackageId** from _Calculator_ to _BadDemoCalculator_. Because the first release pipeline gave me a following warning:

> 'Calculator 1.0.0' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'Calculator 1.0.0' from 'nuget.org'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880

It is caused by selecting this option:

<figure class="wp-block-image">![](/uploads/2019/03/image-4.png)</figure>

It is wise to have this option selected, because typically you won't have conflicts with existing packages. Also, when you use this NuGet feed in your build pipelines, you don't have to use nuget.org explicitly.