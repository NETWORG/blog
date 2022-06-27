---
author: Daniel Klein
title: Fun with Approval Follow up
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
### What are Approvals and what is a Follow up?

Approvals are a nice way of dealing...

Follow up lets you notify Approver that they have a Approval that is waiting for their response.


You can read more on Approvals [here](https://support.microsoft.com/en-us/office/what-is-approvals-a9a01c95-e0bf-4d20-9ada-f7be3fc283d3) and [here](https://powerautomate.microsoft.com/en-us/connectors/details/shared_approvals/approvals/). 

### Trigerring Follow up with a HTTP call

With a bit of Fiddler trickery, we can see what endpoint is called and what body is send. 
The URL we will point at is <br>
<code>https://approvals.teams.microsoft.com/api/sendReminder</code><br>
We can that re-create that in a flow using using the HTTP with Azure AD connector, and its Action Invoke an HTTP request. For this, we need to create a HTTP with Azure AD new connection, that will look like this:
![Connections](/uploads/2022/06/2022-06-27-fun-with-approval-follow-up-02.png)

The final implemantation can look something like this:



### Limitations of Follow ups

Currently, you cannot send follow ups to yourself (funny thing - there is a button for it in Teams, that literary does nothing).

When called by an HTTP call, a follow up for yourself gets made, notifications gets send, however the notifications insides are messed up. 

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

