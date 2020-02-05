---
author: Jan Hajek
title: 'SendGrid, forwarding and DMARC policy'
slug: sendgrid-forwarding-and-dmarc-policy
id: 434
date: '2019-01-23 09:00:03'
categories:
  - English
  - Office 365
  - Security
tags:
  - DKIM
  - DMARC
  - E-mail
  - Microsoft Teams
  - Office 365
  - SendGrid
  - SPF
---

We have recently deployed a strict DMARC policy (p=reject; sp=reject) on our domains. While this adds greater security while sending e-mail and prevents spoofing, we noticed that certain mails forwarded within our organization stopped coming in.

The scenario is, that we quite heavily use Microsoft Teams along with its [e-mail to channel feature](https://support.office.com/en-us/article/send-an-email-to-a-channel-in-teams-d91db004-d9d7-4a47-82e6-fb1b16dfd51e). How it works in Exchange Online is quite simple - we have a Mail Contact (originating from the address generated in Teams - xxxx.test.groups.thenetw.org@emea.teams.ms) created for the Teams address, then a Distribution Group on our domain (say sendgrid@thenetw.org). The contact is then added as a member of the distribution group and everything is nice.

After we tested the policy - [no bad unexpected reports](https://report-uri.com/products/dmarc_monitoring) came in, we decided to tighten up the policy to quarantine the message. Almost instantly, the messages which were being forwarded this way stopped coming to Teams. From the message trace in our EXO, the messages were coming in correctly and being forwarded, but they didn't appear in Teams.

We later identified, that the issue is affecting e-mails sent from SendGrid by our own domain which are being forwarded by Office 365 (eg. we send invoices from SendGrid and a copy goes to Microsoft Teams channel).

As a security precaution, we decided to prevent any sort of automated system, except Office 365 to send e-mails from our primary domain _thenetw.org_, and to have all transactional e-mails sent from a subdomain - _e-mail.thenetw.org_.

When we setted up the domain SendGrid - we also setted up Sender Authentication for domain _e-mail.thenetw.org_. It probably should've occured to us back then, because SendGrid pregenerated a prefix for the domain - _em2509.e-mail.thenetw.org_. What we did was setup SPF for domain _e-mail.thenetw.org_ along with DKIM and all was good - except, when a message is sent, the _smtp.mailfrom_ isn't _e-mail.thenetw.org_ but _em2509.e-mail.thenetw.org_. When being sent directly by SendGrid, everything was working fine, because SendGrid makes you create a CNAME record _em2509.e-mail.thenetw.org_ which points to _u9317285.wl036.sendgrid.net_ which actually has correct SPF set - for SendGrid only however. Now thanks to this, I haven't immediately noticed the fact that the SPF check would fail when the message was being forwarded directly.

The simple solution was to create an SPF record for _em2509.e-mail.thenetw.org_ which contains _v=spf1 include:spf.protection.outlook.com include:sendgrid.net include:u9317285.wl036.sendgrid.net -all_ Office 365 and SendGrid IPs.

Right after setting this up, the e-mails started coming to Microsoft Teams correctly.

We already had a similar SPF record inplace for _e-mail.thenetw.org_ domain, but I was unaware that SendGrid was setting a different Return-Path. While it can be configured while setting up the domain, you are not able to make it the same as the sender domain - so _e-mail.thenetw.org_ as well.

The future and correct solution - the general issue is e-mail forwarding. Forwarding e-mails is pain and generally bad practice. Honestly, I don't understand why Microsoft Teams channels can't have addresses in their tenants - _testgroup+channelid@groups.thenetw.org_ which would prevent this happening in first place. Now the proper solution would be for the Sender Rewriting Scheme to be work with EXO. Actually, the [SRS in EXO](https://blogs.technet.microsoft.com/exchange/2018/06/15/sender-rewriting-scheme-srs-coming-to-office-365/) is being worked on and [per the roadmap](https://www.microsoft.com/en-us/microsoft-365/roadmap?featureid=24056) is being rolled out. Funny thing is, that the rollout was scheduled for Q2 CY2018, and as of now, it is Q1 CY2019 and the status is rolling out still, without any further information. I don't see SRS working in our tenant yet at the time of writing.

Honestly, I hope that Microsoft can finish the SRS deploy soon enough, so that we can revert back to the default SendGrid's configuration.