---
author: Marek Dubsky
title: Troubleshooting 550 5.7.708 error from EXO
slug: >-
  remote-server-returned-550-5-7-708-service-unavailable-access-denied-traffic-not-accepted-from-this-ip
id: 985
date: '2019-10-31 09:00:54'
categories:
  - English
  - Office 365
tags:
  - Exchange Online
  - EXO
  - Office 365
---

From time to time, when you start Office 365 trial with Exchange Online license, after some time you happen to be cut off from email communication and you get this NDR: Remote Server returned 550 5.7.708 Service unavailable. Access denied, traffic not accepted from this IP.

*   <figure>![](/uploads/2019/10/708-screenshot.png)</figure>

It happens when the tenant is in trial version or after you pay for a full license, and it is caused by temporarily used IP addresses which are not trusted, therefore get blocked sometimes.

The solution is very easy, you just need to create a ticket in Office 365 Admin portal of the tenant and support will unblock it.

It takes up to 24 hours to start working but usually it's faster, it can be an hour or even half an hour.

After unlocking, you should consider buying a full license as soon as possible because as long as you are in the trial you may get blocked again.