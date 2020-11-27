---
title: Setting SharePoint item-level permissions to Azure AD Group with API
date: 2020-11-27T15:50:00+02:00
author: Jan Hajek
categories:
  - Microsoft
  - Microsoft Azure
  - Office 365
  - Microsoft Power Automate
  - Microsoft Flow
  - Azure Logic Apps
tags:
  - SharePoint
  - Microsoft Flow
  - Microsoft Power Automate
  - API
  - Azure AD
---

If you want to programmatically set permissions to documents in SharePoint - it is quite simple, just [use Microsoft Graph](https://docs.microsoft.com/en-us/graph/api/driveitem-createlink?WT.mc_id=AZ-MVP-5003178). If you want to set those permissions to list items, it is slightly more complicated than just calling Microsoft Graph.

We have a customer who requires to have very restrictive permissions on items in a SharePoint list. Like mentioned above, this is fairly simple when using document libraries, but when you want to achieve the same with lists... Well, hold your hats, it's more complex.

So first of all, you may want to break inheritance. In order to do that, you can call a nice API endpoint within SharePoint:

```
HTTP POST

_api/web/lists/getByTitle('<list_name>')/items('<item_id>')/breakroleinheritance(copyRoleAssignments=false, clearSubscopes=true)
```

This is obviously optional, because you may just want to add explicit permission to the item, but in most cases, you will want to break it. Also notice, that we break inheritance with `copyRoleAssignments=false` which means that the assignments from upstream item will be discarded.

Next, you need to convert the Azure AD Group to SharePoint's principal. Objects in Azure Active Directory (AAD) are primarily identified by a [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) also referred to as `objectId` or `id`.

You can obtain this ID from [Azure AD Portal](https://aad.portal.azure.com), [Microsoft 365 Admin](https://admin.microsoft.com) (when you open specific group, take the guid from the address bar) or you can search it via [Microsoft Graph](https://graph.microsoft.com) or using Microsoft Flow's [Office 365 Group connector](https://docs.microsoft.com/en-us/connectors/office365groups/#actions) and others.

In order to convert the Group ID to Principal ID (internal SP identifier) you need to call the following endpoint in SharePoint:

```
HTTP GET

_api/web/siteusers('c:0t.c|tenant|<your_aad_group_id>')
```

Simply replace `<your_aad_group_id>` with your group's ID. The object returned is going to be a complex one, so you may want to check the response and JSON parse it, so you can work with it more easily.

Now you have to set the actual permissions:

```
HTTP POST

_api/web/lists/getByTitle('Test')/items('4')/roleassignments/addroleassignment(principalid=<previousrequest['d']['Id']>,roledefid=<your_role_id>)
```

The `<previousrequest['d']['Id']>` is an integer (whole-number) obtained with the previous request for Principal ID.

There are some well-known role IDs, but you can also make a custom one, in case of that, just call `/_api/web/roledefinitions` to get them for your site, or pull it from the URL when modifying the role.

| Permission | Value |
| -- | -- |
| Full Control | 1073741829|
| Contribute | 1073741827 |
| Read | 1073741826 |

All in all, I just wish that Microsoft made it as easy as with Document Libraries, because this is quite obscure...

> Answer also posted to the [Stack Exchange thread](https://sharepoint.stackexchange.com/questions/286524/add-role-assignment-using-office-365-group/287302#287302).