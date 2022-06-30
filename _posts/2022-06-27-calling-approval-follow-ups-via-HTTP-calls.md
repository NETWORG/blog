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

TODO:// Last check

Recently, I encountered a case where I needed to send Follow up via Power Automate Flow, to automatically notify users about pending Approvals. Let's take a look at the possible solution.

### What are Approvals and what is a Follow up?
To quote Microsoft:
>Approvals in Microsoft Teams is a way to streamline all of your requests and processes with your team or partners.

That is the basic gist. You send an Approval, and chosen Approver decides on whatever he wants to actually approve your approval, or reject it. Now, what is a Follow up? Again, let's quote Microsoft:

>You can follow up on approval requests to remind people to take action. You can send a follow-up notification from the Sent list in the Approvals hub, or in the details of the approval itself

Follow up lets you notify Approver that they have an Approval that is waiting for their response that was not yet provided. Quite handy, considering that agenda these days is fully packed and people often forget ?stuff? (and Approves).

You can read more on Approvals [here](https://support.microsoft.com/en-us/office/what-is-approvals-a9a01c95-e0bf-4d20-9ada-f7be3fc283d3) and [here](https://powerautomate.microsoft.com/en-us/connectors/details/shared_approvals/approvals/). 

### Trigerring Follow up with a HTTP call
Now, normally, you would trigger a Follow up from Teams, from this ?beautiful?  button:
![FollowUpButton](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-01.png)
However, what if we wanted to trigger a Follow up a different way, so that we could use it, for example, in Power Automate Flow?
With a bit of Fiddler trickery, we can see what endpoint is called and what body is sent when triggering a Follow up. 
The URL Teams is pointing with a POST is <br>
<code>https://approvals.teams.microsoft.com/api/sendReminder</code><br>
Cool, let's peek into the body of the request:
```json
{
   "ApprovalId":"<guid_of_the_approve>", 
   "Title":"TestNot",
   "Requester":"<name_of_the_requester>",
   "PendingApprovers":[
      "<guid_of_an_approver>"
   ],
   "FlowEnvironment":"<guid_of_the_environment>",
   "ApprovalCreator":0,
   "MessageId":"",
   "ConversationId":"N/A"
}
```
Now, we know how the call looks, and we can re-create this request in a flow. <br>
?First, we will be using the HTTP with Azure AD connector (to ensure authentication)?, and its Action Invoke an HTTP request. For this, we need to create a HTTP with Azure AD connection, that will look like this: <br>
![Connections](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-02.png)

```json
Base Resource URL: https://approvals.teams.microsoft.com/
Azure AD Resource URI (Application ID URI): https://approvals.teams.microsoft.com/
```
Now, we can create an HTTP request. I found that you can cut a few lines from the body so, the request is a bit simpler.
So, the basic shot at the implementation can look like this: <br>
![FirstImplemantation](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-05.png)
Now, we can fill in values from previous steps (eg. from listing All Approvals) and our final implementation can look something like this:<br>
![Final](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-06.png).

We can see that in a correct call, the Output is an object with an ApprovalId.
Also, our Approver gets notified in Teams. 

![Notified](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-04.png)

### Limitations of Follow ups

Currently, you cannot send follow ups to yourself (funny thing - there is a button for it in Teams, that literary does nothing, leading to some [confusions](https://powerusers.microsoft.com/t5/General-Power-Automate/Approval-App-Follow-up-button-does-nothing/td-p/1184418).

When called by an HTTP call, a follow up for yourself gets made, however the notifications insides, are, sadly, messed up. 

![MessedUp](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-03.png)