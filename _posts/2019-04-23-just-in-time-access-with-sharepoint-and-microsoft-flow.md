---
author: Jan Hajek
title: Just In Time Access with SharePoint and Microsoft Flow
slug: just-in-time-access-with-sharepoint-and-microsoft-flow
id: 457
date: '2019-04-23 10:00:02'
categories:
  - Azure
  - Dynamics 365 / CDS / PowerApps
  - English
  - Security
tags:
  - Azure AD
  - Microsoft Flow
  - Power Platform
  - Privileged Identity Management
  - Security
  - SharePoint
---

When you are managing services which deal with customer's data, sensitive information etc. you should never allow users to directly access the data. Instead, you should use some privileged identity management solution. In this article, we are going to look into how to implement this on our own with the use of SharePoint and Microsoft Flow.

Obviously, there are many existing solutions for this (even 3rd party ones like IdentityHQ from SailPoint). You can use Microsoft's native Privileged Identity Management in Azure AD for roles and resources and Just-In-Time access to virtual machines, but if you are a smaller company with lower budget it may be expensive for you. You could also deploy Microsoft Identity Manager which features PIM, however it is rather expensive and if you want to be cloud-only organization, you probably don't want to be deploying ADDS, SharePoint and MIM itself.

In our company we use Groups to secure everything. From accessing projects, through SharePoint, Azure DevOps to Azure Subscriptions. Honestly, I would say that it is well done for the size of our company. However when you have such "project groups" you shouldn't use them for restricting access to production, because there may be guest users etc.

So what we did was that we made a simple portal in SharePoint which allows you to request access to a certain group which than has to be approved by the group owners. The owners are not group members so they have to request access as well. Along with the request, they have to specify the expiration date and business justification. The automatic expiration automatically removes the user from the group afterwards so they loose access to the resources.

![](/uploads/2019/04/screencapture-emea-flow-microsoft-manage-environments-Default-67266d43-8de7-494d-9ed8-3d1bd3b3a764-flows-fb531e60-bebf-4f90-a32c-9ab59c880507-2019-04-19-10_41_54-955x1024.png)

## Download

*   [Download the zip containing SharePoint site template and Flow / Logic App template](/uploads/2019/04/JIT-DEMO.zip)

## Getting started with the solution

Like previously mentioned, we decided to build this solution with SharePoint Online and Microsoft Flow. The solution consists of two SharePoint lists - Groups (projects) and Requests.

1.  [Import SharePoint template](https://docs.microsoft.com/en-us/sharepoint/dev/general-development/save-download-and-upload-a-sharepoint-site-as-a-template#upload-the-site-template-file-to-a-solutions-gallery) and create a sub-site with it
2.  Create Flow / Logic App from the respective template and connect it with the site you created.

Groups list contains the name of the project tha you are going to access eg. _Production Environment 1_, it contains the ObjectId of the specific group in the Azure AD and the maximum membership length in hours - this is important for managing the elevation lifetime.

Requests list is a list for users to create the elevation request itself. It contains a field to select the group, specify the expiration time and a business justification and also the request Status field to know the current state. The rest is taken care of by Microsoft Flow.

## Some additional tips

*   It is actually good to have a policy on naming the JIT groups, for example prefix everything with _JIT-_.
*   This solution doesn't currently support B2B guests.
*   Remember that this solution is rather a Proof of Concept and is **provided with no guarantees**.