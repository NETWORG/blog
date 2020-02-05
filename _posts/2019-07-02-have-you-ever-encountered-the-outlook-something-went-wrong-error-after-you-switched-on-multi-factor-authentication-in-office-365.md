---
author: Marek Dubsky
title: >-
  Have you ever encountered the Outlook \"something went wrong\" error, after
  you switched on Multi-Factor Authentication in Office 365?
slug: >-
  have-you-ever-encountered-the-outlook-something-went-wrong-error-after-you-switched-on-multi-factor-authentication-in-office-365
id: 773
date: '2019-07-02 20:44:58'
categories:
  - English
  - Office 365
tags:
  - Office 365
---

If you have seen the screen below, you should definitely continue reading, because I'm gonna share with you very simple solution.

![](/uploads/2019/07/MA-MFA-.jpg)

When it happened to me for the first time, I was a bit confused - until I found the solution - then I felt silly (LOL). The answer is enabling Modern Authentication.

It will take you 5 minutes if you are skillful in PowerShell and 15 if you are new to PowerShell as I was.

We will use 7 PowerShell commands, from connecting to Exchange Online through enabling Modern Authentication to verifying it and disconnecting.

In case you wanna connect to Exchange Online PowerShell via MFA, you have to first install a modul in Office 365 admin portal.  
To read more click [here](https://docs.microsoft.com/en-us/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell?view=exchange-ps).

Run PowerShell as an Admin and use the following commands:

    Set-ExecutionPolicy RemoteSigned

    $UserCredential = Get-Credential

    $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://outlook.office365.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection

    Import-PSSession $Session -DisableNameChecking

    Set-OrganizationConfig -OAuth2ClientProfileEnabled $true

    Get-OrganizationConfig | Format-Table Name,OAuth* -Auto

    Remove-PSSession $Session

Now, you guys should be all set to use MFA.

I believe, this is just a small delay, comparing to all the trouble, what using MFA can save you from.

And you only have to do it once per tenant!

Below you can find some more info regarding this topic:

*   [https://docs.microsoft.com/en-us/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/connect-to-exchange-online-powershell?view=exchange-ps](https://docs.microsoft.com/en-us/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/connect-to-exchange-online-powershell?view=exchange-ps)
*   [https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/enable-or-disable-modern-authentication-in-exchange-online](https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/enable-or-disable-modern-authentication-in-exchange-online)
*   [https://docs.microsoft.com/en-us/office365/enterprise/modern-auth-for-office-2013-and-2016?redirectSourcePath=%252farticle%252fe4c45989-4b1a-462e-a81b-2a13191cf517](https://docs.microsoft.com/en-us/office365/enterprise/modern-auth-for-office-2013-and-2016?redirectSourcePath=%252farticle%252fe4c45989-4b1a-462e-a81b-2a13191cf517)

I hope this publication will help you and please, feel free to reach out to us, if you have any questions.