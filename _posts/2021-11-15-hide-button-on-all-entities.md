---
author: Ondrej Juda
title: Hide button on all entities
date: "2021-11-12T17:35:00+0200"
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
  - Ribbon Workbench
tags:
  - PowerApps
  - Model-driven Apps
  - Ribbon Workbench
  - Buttons
  - Hide button
---

Lately, I was wondering, if there is a way to hide some buttons through the whole environment. I have been using Ribbon Workbench to hide buttons for specific entities for quite some time, but I did not want to go through all of them and hide there the one button.

This is the time where **Application Ribbons** come in handy. I historically used it to add a button to the Global Tab.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-01.png)

Now we will use it to hide one button on all entities. Let's take for example **Word Templates**.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-02.png)

Go to [make.powerapps.com](https://make.powerapps.com/) on your environment.
Create a new solution in **Solutions**.
Add **Application Ribbons** to the solution.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-03.png)

Open **Ribbon Workbench** in **XrmToolbox**, connect to your environment, and choose the solution you have just created.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-04.png)

Find the button.

> The first row is usually for some globally situated buttons. Subgrid buttons can be found in the second one. We are looking for the form buttons. These are located in the third row. Look for the **Mscrm.Form.{!EntityLogicalName}.MainTab** section.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-05.png)

Hide the button.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-06.png)

Publish the solution.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-07.png)

Now, when you go through different entities, you can see that the button isn't there.

![](/uploads/2021/11/2021-11-15-hide-button-on-all-entities-08.png)
