---
author: matej.samler@thenetw.org
title: >-
  Dynamics 365 Workflow – Why do we activate instead of publishing and why
  lookups lie
slug: >-
  dynamics-365-workflow-why-do-we-activate-instead-of-publishing-and-why-lookups-lie
id: 599
date: '2019-04-08 10:32:17'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Dynamics CRM
  - Workflow
---

Have you always wondered why we need to activate workflows, when every other component, from entities to forms, needs to be saved and published? Is workflow somehow different from other entities in Dynamics? Just to clarify, we are talking about workflows and actions, not business process flows.

Well, yes and no. The definition of a workflow is, of course, stored as records inside of the CDS. CDS uses the entity Workflow (which you won’t find in advanced find) to create records of concrete workflows. This entity can be accessed via the WebApi on the _/workflows_ endpoint.

But if you call this endpoint and see the list of all the workflow records you have in your environment; you may notice that some workflows are there multiple times. That is thanks to the way CDS works with workflows under the hood.

You see, when you create a workflow, CDS creates a “Parent” record of your workflow. This record has all the typical information in it, such as the name, if it’s activated, ownerId and the xaml definition of the logic. Every workflow record also has two single-valued navigation properties: **activeworkflowid** and **parentworkflowid**. As you have probably guessed, the parentworkflowid is set to null, because it itself is the parent.

At the same time, CDS creates another record for the very same workflow. Let’s call it “Active”. This active workflow then gives its id to the parent and sets it as the activeworkflowid, while it gets reference to the its parent.

And here we finally answer the question: Why do we have to activate workflow, instead of publishing it? The answer is simple: Every time you publish something in CDS, you change an existing thing. With workflows, every time you activate a workflow, the CDS creates a new record in your database, and sets it as the active workflow to its parent. This would be useful if it was just a change tracker, but the record gets created even when the workflow is the same.

So what? One record cannot be that bad, even though our customizers probably reactivate workflows dozens of times, it does not matter, because when moving solutions between environments, only the Parent workflow gets packaged, and the children stay in the dev. Instance.

True, this will probably not be a problem for you, unless you start comparing these workflow ids. In practise, let’s say you have an entity “Workflow manager” that is there to watch the runs of his workflow. Typically, you would create an entity, give it a lookup for workflow and then compare the ids in a code activity or plugin

<figure class="wp-block-image">![](/uploads/2019/04/workflow-on-form.png)</figure>

But this is the part where we run into a problem. The record being referenced in the lookup is NOT the same that is used when executing a workflow. The record in the lookup is the Parent record,while the one executed in the async operation (found under owningextentionid) is the active workflow. Meaning that if you wanted to return all async operations of workflow from Workflow managers lookup, you would have to query for parentId, and not the id of the workflow.

<figure class="wp-block-image">![](/uploads/2019/04/workflow.png)</figure>

So, there we have it, this is how CDS handles different versions of workflows, and why you must activate it instead of publishing. Give the endpoint a try and check if you didn’t accidently create dozens of records by reactivating your business logic.