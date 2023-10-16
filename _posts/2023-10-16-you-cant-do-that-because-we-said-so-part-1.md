---
title: You can't do that because "... we said so" - Part 1
date: 2023-10-16T11:00:00+02:00
author: Jan Hajek
categories:
  - Microsoft
  - Security
  - Power Platform
tags:
  - Azure AD
  - Best Practices
  - Authentication
---

This is a long overdue article (probably a series) about how corporate IT security is often forcing "security through obscurity" and how it is not helping anyone. The first in the series is going to address deployments and authentication - and the things we encountered while deploying Power Platform solutions in customer tenants.

<!-- more -->

# The ideal process

Automation is at heart of everything we do in our company. From builds, through testing to deploys. Deployments to production (customer tenants) are done via Azure Pipelines, which then use a shared Service Principal in Entra to authenticate to the customer tenant and deploy the solution to the Dataverse environment.

We utilize a separate tenant which also hosts our production service deployments, so everything is separate from our company's business tenant. Access is granted in a Just-in-Time (JIT) and Just-Enough-Administration (JEA) principles, so that nobody has access unless they request it for a limited time. The service principal (we operate quite a lot of them, but I will focus on the one used specifically for deployments right now) is a multi-tenant app registration which has no scopes or permissions granted to it, since all permissions are configured on the [specific environment](https://learn.microsoft.com/en-us/power-platform/admin/manage-application-users&WT.mc_id=AZ-MVP-5003178).

The service principal is set up with a certificate, which is then used to authenticate via [client_credentials](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow&WT.mc_id=AZ-MVP-5003178) flow. The certificate is stored in [Secure Files in AzDO](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/secure-files?view=azure-devops&WT.mc_id=AZ-MVP-5003178) and access to it is granted only to production pipelines.

We also utilize a separate deployment principal for internal DEV environments, so deployments are truly separated.

Similar situation is with services (like Word converter, e-mail connector, Adaptive Cards and many others which we run). Each is a separate service principal, in case of a connector available for Power Automate, a second, dedicated, service principal for Power Automate as a client is used which is used for user authentication and obtaining the token for the backend service. I admit that this partially [has a flaw](https://github.com/microsoft/PowerPlatformConnectors/issues/596), but since the service principal in Flow is used only for user authentication (not for service-to-service communciation), there is little to no risk of this being abused, unless customer tenant application security is configured wrong - for example, allow to call a custom API with any valid S2S token etc. which should never be allowed! With Power Automate, we previously used the "unofficial" support for client_credentials in custom connectors which did the job, with Microsoft [enabling this functionality officially](https://powerapps.microsoft.com/en-us/blog/public-preview-of-new-custom-connector-enhancements/) without any workarounds (this brings another challenge, but more about that later). 

# What do we have to deal with during deployments

Starting with onboarding, the customer is asked to grant consent to all the required service principals - the one for deployment, and any others which are required by the deployed workload (for custom connectors to work for example). This is a one-time thing. And this is the first problem.

The corporate IT can take weeks or months to consent the principal, requires a bunch of meetings and explaining (usually to different people) and sometimes it feels like it's the first time they heard the "service principal" term. Usually they raise a lot of questions, which are not even relevant and sometimes it feels like the people in charge of security there can't even read Microsoft's own documentation and we have to do all the explaining.

This is both very boring and annoying, and prolongs the entire process.

I also love it when the conversation goes into the "this is not safe, opens up a lot of possible security issues etc." and few minutes after, they say - "yeah, we will give you a service account". Yeah, sure. Give us an interactive service account, which has access god knows where (it can browse the tenant, and [do bunch of other funny things](https://aadinternals.com/aadinternals/)) and requires some more licenses. Then you get a password for the service account, usually not even an enforced MFA or CA policy and you are good to go. Ehm, what?

You're telling me this is more secure than a service principal? I don't think so.

When we engage with the corporate IT, the discussions are all the same. I especially love when it starts steering into the ways of manual deployment... We leverage automation, because we deploy tens of solutions. We want the process to be as automated and human-error prone as possible, and the "offical" corporate IT guidance is - do it manually.

# Why is creating a Service Principal such a big problem?

Sometimes, it's even an issue to get a service principal created. The common reasons are - there is no process for that, we (IT) don't understand service principals, or that the IT is afraid we will have too many permissions in the tenant.

The last is a valid concern. But there are ways! If we are talking about Graph API, there ways, to limit certain permissions - like via [Exchange Online's RBAC](https://learn.microsoft.com/en-us/exchange/permissions-exo/application-rbac), which limits the application scope to a specific group of people. I totally understand that nobody wants to give full mailbox access of every user in their tenant across the globe, just because a branch in one small country needs to use it for a specific application.

The similar goes for SharePoint. The exposure can be controlled via use of `Sites.Selected` scope, further described [here](https://devblogs.microsoft.com/microsoft365dev/controlling-app-access-on-specific-sharepoint-site-collections/).

There are of course scenarios, where you can't limit this, like working with AAD's Group Memberships, in a sync scenario, where you need `GroupMember.ReadWrite.All`. But then, all of this is about customer and supplier trust, which we have estabilished multiple times.

# Security through obscurity and shadow IT

While the corporate IT attempts to fight [shadow IT](https://learn.microsoft.com/en-us/defender-cloud-apps/tutorial-shadow-it?WT.mc_id=AZ-MVP-5003178) which, I ackowledge, is a huge security issue and a nightmare of many, including us, they shouldn't on the other hand actively push us towards doing it.

While I understand that a service account has to be explicitly used in many Power Platform scenarios still, it's becoming less of an issue. We have solution for Power Automate connectors, we have a solution for pipelines, there's a solution for environment management.

I will just briefly touch the service account credentials topic. You usually end up with a username and a password sent via an e-mail. As a partner delivering the solution - where do you store such password? Do you enable MFA on that account? Does the customer even allow MFA on the account? How do you handle access revocation when an employee who had access to the credentials when they leave? Do you rotate the password (which can break a lot of Power Platform connector scenarios)? Just a few things to think about.

Why not let us use service principals, and leverage all the benefits which come with it like auditing and conditional access to further restrict malicious usage?

# My ask to you as an IT decision maker

Please do think twice before you say "no" to creating a service principal or granting consent (and also do think twice before granting it 😊). Sometimes, it makes me feel that we are not on the same side, while we should be striving to innovate and make our users more productive and happy.
