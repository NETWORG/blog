---
author: Ondrej Juda
title: Business rules based on related bpf stage
date: "2022-03-07T08:00:00+0100"
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
  - Business Process Flow
  - Business Rule
tags:
  - PowerApps
  - Model-driven Apps
  - Business Rule
  - Business Process Flow
---

![Form](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-01.png)

Have you ever got into a situation, where you needed to lock or hide some fields when you moved business process flow to the next stage in your model-driven application?

You can use out-of-box functionality in Power Apps called business rule. If you don't have any experience with them, I suggest you to read the Microsoft documentation for [Business Rules](https://docs.microsoft.com/en-us/powerapps/maker/data-platform/data-platform-create-business-rule).

We have an entity record with related business process flow called Remuneration Component. We need to achieve that when the process moves to the second stage, fields on a form would be locked.

The first thing that came to my mind was to create a business rule. The problem was that when I tried to do it from the new UI in [make.powerapps.com](https://make.powerapps.com/), I could not find any way to refer to the related business process flow stage. After some time of searching and clicking, I found out, that when you switch to the classic developer UI, it starts to show the bpf option.

![Business Rule with BPF](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-02.png)

So the simplest way to achieve this condition (for me of course) would be this one:

1. Create a solution and add the entity there.
2. Open the entity and click on the **Switch to classic** button.

| ![Switch to classic](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-03.png)

3. Navigate to the Business Rules section and click on the **New** button.

| ![New button](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-04.png)

4. Open and set the condition to the business process flow stage.

| ![BPF condition](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-05.png)

Once you save the business rule, it does not matter, if you open it from the classic or the new UI. The condition will be there and you can change it however you want.

From here, you can add steps to apply business logic based on a stage of related business process flow.

![Complete business rule](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-06.png)

![Locked form](/uploads/2022/03/2022-03-07-business-rule-on-related-bpf-stage-07.png)
