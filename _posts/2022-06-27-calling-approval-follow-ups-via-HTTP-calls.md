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
  - Power Automate
  - Approval
  - Microsoft Flow
  - Follow up
  - Approval Follow up
---

Recently, I encountered a case where I needed to send a Follow up via Power Automate Flow, to automatically notify users about pending Approvals. Let's take a look at the possible solution.

### What are Approvals and what is a Follow up?
To quote Microsoft:
>Approvals in Microsoft Teams is a way to streamline all of your requests and processes with your team or partners.

That is the basic gist. You send an Approval, and chosen Approver decides on whatever he wants to actually approve your approval, or reject it. Now, what is a Follow up? Again, let's quote Microsoft:

>You can follow up on approval requests to remind people to take action. You can send a follow-up notification from the Sent list in the Approvals hub, or in the details of the approval itself

Follow up lets you notify the Approver that they have an Approval waiting for their response that was not yet provided. Quite handy, considering that the agenda these days is fully packed.

You can read more on Approvals [here](https://support.microsoft.com/en-us/office/what-is-approvals-a9a01c95-e0bf-4d20-9ada-f7be3fc283d3) and [here](https://powerautomate.microsoft.com/en-us/connectors/details/shared_approvals/approvals/). 

### Trigerring Follow up with an HTTP call
Now, normally, you would trigger a Follow up from Teams, from this button, located on the tab of a given Approval:
![FollowUpButton](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-01.png) <br>
However, what if we wanted to trigger a Follow up a different way, so that we could use it, for example, in Power Automate Flow?
With a bit of Fiddler trickery, we can see what endpoint is called and what body is sent when triggering a Follow up. 
The URL Teams is pointing at with a POST method is <br>
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
We will be using the HTTP with Azure AD connector (to ensure authentication), and its Action Invoke an HTTP request. For this, we need to create an HTTP with Azure AD connection, that will look like this: <br>
![Connections](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-02.png)

```
Base Resource URL: https://approvals.teams.microsoft.com/
Azure AD Resource URI (Application ID URI): https://approvals.teams.microsoft.com/
```
Now, we can re-create our HTTP request. I found that you can cut a few lines from the body so, in the end, the request is a bit simpler.
The basic shot at the implementation can look like this: <br>
![FirstImplemantation](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-05.png)
Now, we can fill in values from previous steps (e.g. from listing All Approvals and obtaining needed properties) and our final implementation can look something like this:<br>
![Final](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-06.png).

We can see that in a correct call, the Output is an object with an ApprovalId.
Also, our Approver gets notified in Teams. 

![Notified](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-04.png)

### Limitations of Follow ups

Currently, you cannot send follow ups to yourself (funny thing - there is a button for it in Teams, that literary does nothing, leading to some [confusions](https://powerusers.microsoft.com/t5/General-Power-Automate/Approval-App-Follow-up-button-does-nothing/td-p/1184418).

When called by an HTTP call, a follow up for yourself gets made, however, the notifications insides, are, sadly, messed up. 

![MessedUp](/uploads/2022/06/2022-06-27-calling-approval-follow-ups-via-HTTP-calls-03.png)