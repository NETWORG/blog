---
author: frantisek.capek@thenetw.org
title: 'Dynamics 365 - Emails: Mailboxes & Queues'
slug: dynamics-365-mailboxes-queues
id: 972
date: '2019-10-29 09:00:36'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
  - Office 365
tags:
  - Dynamics 365
  - Dynamics CRM
  - email
  - mailbox
  - queue
---

Have you ever struggled with the settings in Dynamics CRM when you were setting up mailboxes and queues? Right.

I guess you already went through this once or twice, especially if you focus on Customer Service, but if you'd ever need to do it again, I'm here to hopefully save you some time.

## User mailboxes

This is pretty straight forward, you simply need to approve the email address and enable the mailbox by running a test. The only important thing is that you should decide what synchronization method you want to use.  
_I suggest you use the server-side synchronization method for incoming and outgoing emails and Outlook for appointments, contacts, and tasks._  
If any user mailbox is not in the CRM, just check if the user has O365 license.

## Queues

In case we need to track emails from a shared mailbox, we should create a queue. When creating a queue, you should create a proper name and fill in the incoming email address. Important thing is the ownership/access. We need to set it to private, so only the agents assigned to this queue will see its content. After you created the queue, go ahead and enable the mailbox.

<div class="wp-block-image">

<figure class="aligncenter">![This is an example of a queue for shared mailbox support@thenetw.org in Dynamics CRM.](/uploads/2019/10/image.png)

<figcaption>This is an example of a queue for shared mailbox support@thenetw.org</figcaption>

</figure>

</div>

Regarding the security, we need to create owner team and assign the queue to this team. After you create the record, assign the users (agents) to this team. Then head back to your queue and assign it to the new created owner team.  
_There may be an exception thrown, if so, then just check the security roles on the team and assign it the base CDS user security role._

Tip: Also consider enabling the App for Outlook for the users, otherwise you should be all set at this point.

Congratulations. You did it!