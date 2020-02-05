---
author: jan.kostejn@thenetw.org
title: Override the default open behavior of data rows in an entity-bound grid
slug: override-the-default-open-behavior-of-data-rows-in-an-entity-bound-grid
id: 408
date: '2018-12-04 14:58:18'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - CDS
  - powerplatform
  - subgrid
---

Hello there, fellow power platform customizers/developers! Have you tried to override the default open behavior in subgrid and failed?   What I’ve found in my research: [Documentation by Microsoft says](https://docs.microsoft.com/en-us/dynamics365/get-started/whats-new/customer-engagement/new-in-version-9-for-developers#override-the-default-open-behavior-of-data-rows-in-an-entity-bound-grid): “You can now create a command definition for an entity with Mscrm.OpenRecordItem as the value of the Id attribute (<CommandDefinition> (RibbonDiffXml)), and define custom action for the command <Actions> (RibbonDiffXml). Customer Engagement will look for this command Id for an entity when you try to open a record from the entity-bound grid, and if present, will execute the custom action instead of opening the entity record (default behavior).” IMPORTANT NOTE: This feature is supported only for Unified Interface. Did this. No luck. What now? Google… I found only one related article/discussion: [https://community.dynamics.com/crm/f/117/t/254488](https://community.dynamics.com/crm/f/117/t/254488) Where you can get more concrete visualization about how it should look like. But nobody has tried it on the new UUI.   So, I had this snippet:

<pre class="lang:xhtml decode:true" title="First draft"><RibbonDiffXml>
    <CustomActions />
    <Templates>
        <RibbonTemplates Id="Mscrm.Templates"></RibbonTemplates>
    </Templates>
    <CommandDefinitions>
        <CommandDefinition Id="Mscrm.OpenRecordItem">
        <EnableRules />
        <DisplayRules />
        <Actions>
            <!--<JavaScriptFunction FunctionName="openDocument" Library="$webresource:tntg_openbehaviorofdatarows" />-->
            <Url Address="https://www.talxis.com/" WinMode="0"></Url>
        </Actions>
        </CommandDefinition>
    </CommandDefinitions>
    <RuleDefinitions>
        <TabDisplayRules />
        <DisplayRules />
        <EnableRules />
    </RuleDefinitions>
    <LocLabels />
</RibbonDiffXml></pre>

This <RibbonDiffXml> element is in customization.xml for each Entity.   Although we followed every step of documentation, this wasn’t working. I was stuck, because no one had the solution on the internet and that’s why I’m writing this blogpost, so that you don’t find yourself in similar situation.   Solution: What documentation doesn’t say (written on 2018/12/04) is that you need to add another element to your customization.xml. You need to add <CustomActions>. So, the snippet should look like this:

<pre class="lang:xhtml decode:true" title="Final version"><RibbonDiffXml>
    <CustomActions>
        <CustomAction Id="New_Mscrm.SubGrid.connection.MainTab.Actions.Controls.CustomAction" Location="Mscrm.SubGrid.connection.MainTab.Actions.Controls._children" Sequence="150">
        <CommandUIDefinition>
            <Button Command="Mscrm.OpenRecordItem" Id="Mscrm.OpenRecordItem" />
        </CommandUIDefinition>
        </CustomAction>
    </CustomActions>
    <Templates>
        <RibbonTemplates Id="Mscrm.Templates"></RibbonTemplates>
    </Templates>
    <CommandDefinitions>
        <CommandDefinition Id="Mscrm.OpenRecordItem">
        <EnableRules />
        <DisplayRules />
        <Actions>
            <!--<JavaScriptFunction FunctionName="openDocument" Library="$webresource:tntg_openbehaviorofdatarows" />-->
            <Url Address="https://www.talxis.com/" WinMode="0"></Url>
        </Actions>
        </CommandDefinition>
    </CommandDefinitions>
    <RuleDefinitions>
        <TabDisplayRules />
        <DisplayRules />
        <EnableRules />
    </RuleDefinitions>
    <LocLabels />
</RibbonDiffXml>
</pre>

  Now import your solution and don’t forget to publish all customizations…   Happy PowerPlatforming!