---
author: Matej Samler
title: Custom Connector suddenly unauthorized - Fix/Workaround
date: '2020-11-19T14:50:00+0200'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - Microsoft Flow
  - Power Automate
---

You may have encountered an error recently with a Flow in Power Automate that uses custom connectors that require some sort of authentication, in which the flow stops working all of a sudden, giving you a 401 unauthorized error. 

![](/uploads/2020/11/unauthorized.png)

Sadly, this is a bug on Microsofts site. It can happen when you deploy a new version of your flow.

Luckily, there is a quick and easy workaround.):

1) Open the custom connector in edit

2) Toggle Swagger editor, **CLICK ANYWHERE IN THE YAML DEFINITION** and then hit update. (I have tried it without the click and it seems like that doesn't refresh it) You don't need to change anything, just click inside

3) After saving, go to the "Test" section and input random data. If this method didn't work, you would get a 401 (unauthorized), but if it did, you should get 500 error. You could also input some valid data for a 200, but I'll leave that up to you.

![](/uploads/2020/11/UnauthorizedConnector.gif)

Thankfully, custom connectors do not have solution layering, so this will not be an unmanaged change. After doing this, you can resubmit your failed flow and it should work as expected.

