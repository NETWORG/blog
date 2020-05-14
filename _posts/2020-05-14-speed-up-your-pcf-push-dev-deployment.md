---
author: Jan Hajek
title: Speed up your PCF push dev deployment
date: '2020-05-14 12:30:00'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PCF
  - PowerApps
---

When you develop controsl with PCF tooling, you can make use of [`pac pcf push`](https://powerusers.microsoft.com/t5/Power-Apps-Pro-Dev-ISV/pac-pcf-push-FAQ/td-p/356312) command which will push the components to your environment, so you can test it.

Upon push, it creates a new temporary solution and uploads it to the target environment. It also publishes all the customizations and so on, which takes quite a long time. In some previous version (currently using v1.2.6) the tooling was able to do incremental push, but for some reason, the functionality seems to be off by default.

The incremental push looks into the target environment, checks if the `ControlManifest.*.xml` has changes, and if not, it simply updates the underlying generated web resources only, which saves you a lot of time. It also comapres the respective fiels, so unless you made a modification, it won't end up being uplaoded, again, saving your time.

Since this feature is not enabled by default and there are not many references, I went ahead and tried to enable it myself (thank you [ILSpy](https://github.com/icsharpcode/ILSpy)). Turns out the incremental push is still i nthe tooling, it's just not enabled for some reason. So in order to enable it, you have to do the following:

1. Go to: `%LOCALUSERPROFILE%\Microsoft\PowerAppsCLI\` open the CLI folder and the tools folder in it (for me it is `Microsoft.PowerApps.CLI.1.2.6\tools` at the time of writing)
1. Open `featureflags.json` in your favorite editor ([VS Code](http://code.visualstudio.com/))
1. And add the following flag:
  ```json
  "verbPcfPushIncremental": "on",
  ```
1. And you are done

Just for reference, my `featureflags.json` look like this at the time of writing:

```json
{
  "verbAuth": "on",
  "verbOrg": "on",
  "verbOrgEntity": "off",
  "verbOrgQuery": "off",
  "verbSolutionImport": "off",
  "verbSolutionExport": "on",
  "verbSolutionClone": "on",
  "pcfVirtualControl": "on",
  "pcfObjectType": "off",
  "pcfPropertyDependencies": "off",
  "verbPlugin": "on",
  "verbPcfPush": "on",
  "verbPcfPushIncremental": "on",
  "verbPortal": "on"
}
```

Hope it saves your time, enjoy!