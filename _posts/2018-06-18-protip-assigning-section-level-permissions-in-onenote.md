---
author: Jan Hajek
title: 'Protip: Assigning section level permissions in OneNote'
slug: protip-assigning-section-level-permissions-in-onenote
id: 179
date: '2018-06-18 09:00:09'
categories:
  - English
  - Office 365
tags:
  - OneNote
  - Permissions
  - Protip
  - SharePoint
---

When you have a OneNote notebook shared with an entire group or site in SharePoint (or with few people in OneDrive for Business) you might want to be able to set permissions on a section or section-group level. While this functionality isn't for some reason available directly from the UI, it is definitely possible. Read on to learn how! Basically, when you create a OneNote notebook in a document library, what really happens is that a folder is created there with the notebook's name and then the sections (_.one_) files are placed into it. Section groups are represented as folders as well with _.one_ files as sections inside (you can learn more details [here](http://itgroove.net/oh365eh/2014/05/13/onenote-notebook-files-folders-work/)).

Now to the permission part. [OneNote API](https://msdn.microsoft.com/en-us/office/office365/howto/onenote-manage-perms) gives us the ability to modify the permissions through RESTful calls. This functionality is what OneNote Class Notebooks seem to use on the background. This is great, but we would like to use the user interface right? So the easiest way is to navigate to the specific document library, where the file is stored, for example:

`https://thenetworg.sharepoint.com/teams/int0005/SiteAssets/Forms/AllItems.aspx`

Next, you need to navigate into its respective folder (if the notebook is in a folder itself). Then you simply modify the URL to contain the notebook's name, since it is a folder anyways: 

_https://thenetworg.sharepoint.com/teams/int0005/SiteAssets/Forms/AllItems.aspx?id=%2F**teams**%2F**int0005**%2F**SiteAssets**%2F**Site**%20**Notebook**_

Notice the **bold** parts of the address which represent the path to the library, the _%2F_ is just URL escaped / (slash). 

![/uploads/2018/06/onenote-sharepoint-permissions-300x130.png](/uploads/2018/06/onenote-sharepoint-permissions.png)

From there, you can easily stop inherting the permissions and assigning your own. This works both in regular SharePoint sites and Office 365 Groups as well.