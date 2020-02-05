---
author: Jan Hajek
title: Lost access to Linux VM in Azure and no idea what to do? Check this out...
slug: lost-access-to-linux-vm-in-azure-and-no-idea-what-to-do-check-this-out
id: 624
date: '2019-04-29 09:00:57'
categories:
  - Azure
  - English
  - Security
tags:
  - Authentication
  - Azure Active Directory
  - Azure AD
  - Linux
  - PAM
  - SSH
---

Last week, we have hit a really interesting issue with our Linux machines in Azure. We "somehow" (will be explained later in the post) managed to get completely locked out of the machine, not even Serial Console could have been used to login. After bunch of time spent by investigating the situation, we managed to get it resolved.

While tryong to do some regular maintenance work, I noticed I couldn't sign in to a Linux VM running in Azure using [Azure AD Login for Linux extension](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/login-using-aad). So as a fall back, we have a regular account with SSH key setup for login - didn't work either. Always ended with following error:

    Connection closed by XXX.XXX.XXX.XXX

So my initial thought was that something is wrong with SSH on the server, so I went ahead and [reset the configuration via Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/vmaccess#restart-ssh). Still, nothing, same behavior with Connection closed.

So while getting more and more nervous, I decided to go ahead and use the [Serial Console](https://docs.microsoft.com/en-us/azure/virtual-machines/troubleshooting/serial-console-linux). First, I had to create a user with an actual password, which is [quite easy thanks to Microsoft Azure's tooling](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/vmaccess#create-an-administrativesudo-user). Next using that account I tried logging via serial console. Interestingly I ended up in a loop which kept prompting me for my username, didn't even get to typing the password.

That sort of pointed me to the idea that something must be wrong with login rather than SSH. So after some investigation, I figured that there is something wrong with the PAM configuration.

The only thing that modified the PAM (which I was aware of) was the AAD Login for Linux extension. So I generally tried to uninstall it from the portal - but for some reason, the uninstallation didn't go through. Even tho the serial console listed the fact that the extension has been removed. In order to remove it properly I needed the access to the VM or the filesystem at least.

Being unable to access the machine though, how to handle that? Download the VHD, try to fix it somewhere and upload it back? That was like the very last thing I wanted to do because that would involve downtime and eventually a lot of work.

Instead, I used the [Run Command](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/run-command) feature of Azure. Run Command basically allows you to run non-interactive commands remotely on the machine in the context of a root user. While [it is technically "backdoor"](https://raymii.org/s/blog/Linux_on_Microsoft_Azure_Disable_this_built_in_root_access_backdoor.html) it actually saved me from a lot of downtime.

First off, I found this wonderful article about [how the AAD Login for Linux extension works](https://kvaes.wordpress.com/2018/07/17/taking-a-look-under-the-hood-of-the-linux-vm-authentication/). As you can see in the article, it actually modifies the PAM configuration.

I found out that the extension is actually installed as a Debian package on the machine and the package is [here](http://packages.microsoft.com/ubuntu/18.04/prod/pool/main/a/aadlogin/). From that package (just use 7-Zip and open it as archive), I managed to pull the uninstall script and then execute it via Run Command on the machine - which effectively removed everything.

After that, I was able to login to the machine again via both SSH and Serial Console. I did the same action on the rest of the affected machines and voila!

I cannot really however explain what went wrong in the first place and why it broke on all our Linux machines. After that, I just reinstalled the AAD Login for Linux extension to the machines and everything was fine again.

I personally think that it may have something to do with two releases [being released after one another](http://packages.microsoft.com/ubuntu/18.04/prod/pool/main/a/aadlogin/). Maybe someone forgot to mention a bug in the release notes? Or maybe something just went wrong (on bunch of machines during the same time window). Either way, the extension is still in Preview (beware when using it!) and it still pretty much rocks!