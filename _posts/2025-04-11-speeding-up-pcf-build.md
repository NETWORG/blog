---
title: Speeding up PCF build
date: 2025-04-11T12:30:00+02:00
author: Jan Hajek
categories:
  - Microsoft
  - Microsoft Power Platform
tags:
  - Power Apps component framework
  - Rush
link: https://hajekj.net/2025/03/01/speeding-up-pcf-build/
---

At [NETWORG](https://www.networg.com) we have been extensively using [Power Apps component framework](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview) which allows us to build custom UI components for Power Apps. Our largest repo with controls (which we continuously update and re-use across customers) has over 47 controls. The repo is setup with [Rush framework](https://rushjs.io/) to streamline dependency management (more on that topic in another article). And the build for each control in CI (Azure DevOps) averages for 1:30 minutes (the build altogether takes 70 minutes).


[Full Article](https://hajekj.net/2025/03/01/speeding-up-pcf-build/)