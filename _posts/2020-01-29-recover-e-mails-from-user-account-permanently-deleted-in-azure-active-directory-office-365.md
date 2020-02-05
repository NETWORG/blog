---
author: Jakub Wyczanski
title: >-
  Recover e-mails from user account permanently deleted in Azure Active
  Directory (Office 365)
slug: >-
  recover-e-mails-from-user-account-permanently-deleted-in-azure-active-directory-office-365
id: 1074
date: '2020-01-29 09:00:00'
categories:
  - Azure
  - Office 365
tags:
  - Azure Active Directory
  - Office 365
---

Picture this: You need to solve a ticket, where a user needs to be recovered after being deleted. Not a big deal, just access the deleted users from the Office 365 Admin Center or Azure Active Directory, right? Well, no in this case scenario. The user was permanently deleted in the Azure Azure Active Directory so the user cann be only added from scratch, but what about the e-mails? Is this a lost game? Luckily not. The hard deleted user account still may be listed in the Exchange Online Admin Center as a deleted mailbox and will stay there for ongoing 30 days.

So – if this condition is met, the content of the still soft deleted mailbox can be recovered. How? Not as straightforqard at first glance, because if you try it from EXO Admin center, it fails and you will be greeted with a suggested cmdlet in Powershell to execute, that looks like this:

_New-MailboxRestoreRequest_

So this is what I did and get a prompt for the source e-mail and the target e-mail to move the content of the deleted mailbox to, but I got an error, again.

Upon further research I found the cmdlets that lead me to the solution.

First I double-checked if tej mailbox is still listed as soft-dleted with the cmdlet below:

_Get-Mailbox -SoftDeletedMailbox_

Afterwards I enterd the command below to move the content of the soft deleted mailbox to the admins mailbox to anspecified folder:

_New-MailboxRestoreRequest -SourceMailbox deletedmailbox@domain.com* -TargetMailbox activetargetmailbox@domain.com** -TargetRootFolder deletedmailbox -restore -AllowLegacyDNMismatch_

* deletedmailbox@domain.com – deleted user

** [activetargetmailbox@domain.com](mailto:activetargetmailbox@domain.com) - active user

Then checked the status with the command below:

_Get-MailboxRestoreRequest_

As soon as the status was done it took some time and the folder with e-mails showed up i Outlook.

So I hope you will be able to sort out that kind of issues with this post. Feel free to comment.