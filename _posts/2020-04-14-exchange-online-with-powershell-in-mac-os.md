---
author: Jakub Wyczanski
title: Exchange Online with PowerShell in Mac OS
date: 2020-05-01T09:00:00+02:00
categories: 
- Exchange Online
- PowerShell
- Mac OS
Tags:
- Mac
- Powershell
- Exchange
---

Exchange online with PowerShell in Mac OS

Let’s say, your company has no Windows PC and has no interest of buying one in the near future, but there are tasks you need to setup ASAP in Exchange Online, that can only be done via PowerShell. Bad luck?

Luckily not quite. There are two options to choose from. One is to subscribe Azure Cloud and use the Azure shell, which is not a very affordable solution to say the least, but it is definitely easier (aside a VM with Windows).

You could save the cost, if you need PowerShell temporarily and opt in for the free trial, that will run 30 days, but her is a second option, Microsoft has in development, that I will cover here.

Theoretically Microsoft did describe it pretty well in the Docs, but in practice it isn’t as straightforward. To sum it up, there are 3 crucial steps:

1. Choose the way you install the PowerShell Core (Homebrew or a release from GitHub)
2. Install Homebrew (or XCode and Mac Ports, but this option doesn’t work for connecting with EXO)
3. Install OpenSSL

These steps are well covered in the official article in Microsoft Docs:
https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-macos?view=powershell-7

The thing is, that it doesn’t work that simple, if you want to use the Exchange Online PowerShell Module to connect to EXO. Let me sum up how it went for me.

As I followed the official guidelines, installed Homebrew and PowerShell, I went on and tried connecting, her is what happen as the output after establishing the connection:

 ![Screenshot 1](/uploads/2020/04/EXO_Powershell_Mac_Error.png)

Well, WSMan is a tool present on Windows, so something isn’t right. OK. Time to search for solutions. After googling, instead of reading the article to its end, I eventually found the necessity to install OpenSSL, but I still got the same error. 

As it turned out, the issue was known for longer and various solutions was suggested on GitHub bug reports, so I finally found the best way to overcome the error above. Here is the scenario:

-	First of all, you install Homebrew, It is a crucial point to actually make use of PowerShell core and successfully connect to EXO
-	Next you install the PowerShell Core, method is not important, the newest stable build from Homebrew and other releases should work fine
-	The crucial thing Homebrew is really important for, is the version of OpenSSL, i.e. you need to install the version 1.02. Let’s assume, OpenSSL is already installed, just go the following path:
brew uninstall openssl; to remove the most current stable release

brew uninstall openssl; (to ensure nothing was left)

brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb

And afterwards it will work… Not yet, for me at least. 
All depends, if you have Security Standards enabled, if you do, you will need to disable it to use Baseline policies, or the authentication will not work. I had them activated, which forces admins to use MFA, even if disabled and because of that, you can’t use basic authentication, which in this scenario is the only option, because the PowerShell module for MFA is only Windows compatible. 
The error might be surprising, as it isn’t the same on Mac OS and Windows. On the latter I saw “Access is denied”, whereas on Mac OS it was “Basic authentication failed for user”, but after several attempts I found the only solution is to go back to Baseline Polices as mentioned. No PowerShell trickery in Msol Shell will help, believe me, but it was fun to see it working on a Mac.

 ![Screenshot 2](/uploads/2020/04/EXO_PowerShell_Mac_working.png)

DISCLAIMER: Deactivating Security Standards are only recommended temporarily, so if you need to work with the PowerShell Core for longer, there is an option to create a Conditional Access Policy, if you consider upgrading to a subscription that has Azure Active Directory Premium included (Like EMS, Microsoft 365 E3 or E5) or purchase it separately as an add-on.

Although Microsoft wants to be more cross-platform, it is obvious, it is still in quite early development. Especially, when one needs to go against their recommendations and go back to legacy authentication method or upgrade the license. If it was for me, I’d add Azure Shell as an add-on license or work on getting the PowerShell Module for MFA to be cross-platform. There is also an option of importing the Exchange Online V2 Module, but the connection fails overall, and MFA uses Windows Forms to sign in, so her we have a no-go as well.

I hope, you find this helpful and interesting, feel free to comment.
