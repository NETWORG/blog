---
author: Daniel Klein
title: Calling Approval Follow ups via HTTP calls
date: '2022-06-27T15:00:00+0200'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
  - Teams
  - Approvals
tags:
  - Teams
  - PowerApps
  - Approval
  - Follow up
  - Approval Follow up
---

TODO:// Overall clean, try calling HTTP call on different env (missing flowenv in request)
Intro
Outro
Screen of request
Removing personal info
### What are Approvals and what is a Follow up?
To quote Microsoft:
>Approvals in Microsoft Teams is a way to streamline all of your requests and processes with your team or partners.

That is the basic gist. You send an Approval, and chosen Approver decides on whatever he wants to actually approve your approval, or reject it. Now, what is a Follow up? Again, lets quote Microsoft:

>You can follow up on approval requests to remind people to take action. You can send a follow-up notification from the Sent list in the Approvals hub, or in the details of the approval itself

Follow up lets you notify Approver that they have a Approval that is waiting for their response that was not yet provided. Quite handy, considering that agenda these days is fully packed and people often forget stuff (and Approves).

You can read more on Approvals [here](https://support.microsoft.com/en-us/office/what-is-approvals-a9a01c95-e0bf-4d20-9ada-f7be3fc283d3) and [here](https://powerautomate.microsoft.com/en-us/connectors/details/shared_approvals/approvals/). 

### Trigerring Follow up with a HTTP call
Now, normally, you would trigger a Follow up from Teams, from this beatiful button:
![FollowUpButton](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-01.png)
However, what if we wanted to trigger a Follow up a different way, so that we could use it, for example, in PowerAutomate Flow?
With a bit of Fiddler trickery, we can see what endpoint is called and what body is send when triggering a Follow up. 
The URL Teams is pointing at is <br>
<code>https://approvals.teams.microsoft.com/api/sendReminder</code><br>
Cool, lets peak into the body of the request:
```json
{
   "ApprovalId":"af1a3f60-ed79-4b4c-97f3-5de236a37006", 
   "Title":"TestNot",
   "Requester":"Daniel Klein",
   "PendingApprovers":[
      "17b1abd4-f100-4ca5-b930-8c9a25b36a55"
   ],
   "FlowEnvironment":"Default-67266d43-8de7-494d-9ed8-3d1bd3b3a764",
   "ApprovalCreator":0,
   "MessageId":"",
   "ConversationId":"N/A"
}
```
Now, we know how the call looks, and we can re-create this request in a flow. <br>
First of, we will be using the HTTP with Azure AD connector (to ensure authetication), and its Action Invoke an HTTP request. For this, we need to create a HTTP with Azure AD connection, that will look like this: <br>
![Connections](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-02.png)

```json
Base Resource URL: https://approvals.teams.microsoft.com/
Azure AD Resource URI (Application ID URI): https://approvals.teams.microsoft.com/
```
Now, we can create an HTTP request. I found that you can cut few lines from the body so, the request is a bit simpler.
The implemantation can look something like this:

Aand our Approver gets notified in Teams. 

### Limitations of Follow ups

Currently, you cannot send follow ups to yourself (funny thing - there is a button for it in Teams, that literary does nothing, leading to some [confusions](https://powerusers.microsoft.com/t5/General-Power-Automate/Approval-App-Follow-up-button-does-nothing/td-p/1184418).

When called by an HTTP call, a follow up for yourself gets made, notifications gets send, however the notifications insides, are, sadly, messed up. 

![MessedUp](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-03.png)


https://powerusers.microsoft.com/t5/General-Power-Automate/Approval-App-Follow-up-button-does-nothing/td-p/1184418

{
   "ApprovalId":"@{items('For-Each-Approval')?['Name']}",
   "Title":"@{items('For-Each-Approval')?['Properties']?['Title']}",
   "Requester":"@{items('For-Each-Approval')['Properties']['Principals']?[0]?['DisplayName']}",
   "PendingApprovers":@{items('For-Each-Approval')?['Properties']?['Approvers']}
}


*Method
POST
*Url of the request
https://approvals.teams.microsoft.com/api/sendReminder
Headers

Enter key

Enter value

