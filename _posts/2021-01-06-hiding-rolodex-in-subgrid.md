---
author: Ondrej Juda
title: Hiding rolodex in subgrid
date: '2021-01-06T18:20:00+0200'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - Model-driven Apps
---

If you want to hide rolodex on your subgrid (the bar on the bottom with whole alphabet), this post is for you. While working with dashboards, one line caught my eye in their XML definition - **\<EnableJumpBar\>false\</EnableJumpBar\>**.

```xml
<control id="Component6bd36f6" uniqueid="{37de83d6-ebb8-a143-942d-0455a095150d}" classid="{E7A81278-8635-4d9e-8D4D-59480B391C5B}" indicationOfSubgrid="true">
  <parameters>
    <TargetEntityType>account</TargetEntityType>
    <ChartGridMode>Grid</ChartGridMode>
    <EnableQuickFind>true</EnableQuickFind>
    <EnableViewPicker>false</EnableViewPicker>
    <EnableJumpBar>false</EnableJumpBar>
    <RecordsPerPage>9</RecordsPerPage>
    <HeaderColorCode>#F3F3F3</HeaderColorCode>
    <ViewId>{ED3DA4FB-E954-4936-BA14-9E813F1303DF}</ViewId>
    <IsUserView>false</IsUserView>
    <ViewIds>{6CFFA2D3-69BC-48E1-A265-BF7363CFFEB6}</ViewIds>
    <AutoExpand>Fixed</AutoExpand>
    <VisualizationId />
    <IsUserChart>false</IsUserChart>
    <EnableChartPicker>false</EnableChartPicker>
    <RelationshipName />
  </parameters>
</control>
```

As I noticed the one subgrid did not have rolodex on it. So I tried to copy the line from this dashboard subgrid definition to another subgrid definition and it worked as expected. Rolodex from the other subgrid disappeared.

![](/uploads/2021/01/2021-01-06-hiding-rolodex-in-subgrid-01.png)

To achieve the same result you will have to export solution which contains the form where you want to hide rolodex. After unpacking solution, you'll have to find the form xml file and open it. You want to locate the subgrid there and pass this line inside it's definition - **\<EnableJumpBar\>false\</EnableJumpBar\>**. 

```xml
<control id="contactssubgrid" classid="{E7A81278-8635-4D9E-8D4D-59480B391C5B}" indicationOfSubgrid="true">
  <parameters>
    <TargetEntityType>contact</TargetEntityType>
    <ViewId>{6E5D52F1-0012-EB11-A813-000D3A4A162C}</ViewId>
    <ViewIds>{7B0D226E-46D9-469A-8F48-6ECD3E724447},{6E5D52F1-0012-EB11-A813-000D3A4A162C}</ViewIds>
    <EnableViewPicker>false</EnableViewPicker>
    <RelationshipName>contact_customer_accounts</RelationshipName>
    <EnableJumpBar>false</EnableJumpBar>
  </parameters>
</control>
```

After packing the solution and importing it back, the rolodex will dissappear and your subgrid will look like this.

![](/uploads/2021/01/2021-01-06-hiding-rolodex-in-subgrid-02.png)
