---
author: Frantisek Capek
title: Dynamics 365 - Custom Dialogs in UCI - Part 3
slug: dynamics-365-custom-dialogs-in-uci-part-3
id: 689
date: '2019-06-10 08:00:58'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - Dynamics 365
  - Power Platform
---

You may wonder why is this part 3 when there is no part 1 and part 2\. The reason is simple. Bob Guidinger already covered the introduction to the custom dialogs well enough and you can read about it [here](https://bguidinger.com/blog/custom-dialog-boxes-part-1) in his blog post. Please, do so, I'm going to refer to it quite often.

When I was reading the post, I immediately wanted to try it out in my own environment. However, the blog post doesn't cover much about more complex ways of using custom dialogs. So I decided to deep dive into this topic.

I gave myself a task to create custom dialog replicating the one that shows up when user wants to close an opportunity as lost.

<div class="wp-block-image">

<figure class="aligncenter is-resized">![](/uploads/2019/06/Close-Opportunity-as-Lost.png)

<figcaption>Close Opportunity as Lost system dialog.</figcaption>

</figure>

</div>

I started by making custom Customizations.xml file, where I defined the custom dialog, inside the "Dialogs" element. If you don't know how to do it, you can follow the instructions in Bob's blog post.

After that, I added the first field, which is the Status Reason field. It's an OptionSet data type field, which is defined by a global option set. _Note that you cannot use already existing fields, you have to always create new field and set the values equal to the existing field._

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/3dc8de50375696ffedf9f872d9e79b5f#file-Customizations-xml">View this gist on GitHub</a></noscript>

</div>

You can see the global option set below.

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/3dc8de50375696ffedf9f872d9e79b5f#file-OptionSet-xml">View this gist on GitHub</a></noscript>

</div>

The most important thing about this is the **classid** value. There is only one class id for each data type in Dynamics. You can see the list of data types and their proper class id below.

<table class="wp-block-table">

<tbody>

<tr>

<td>Single Line of Text</td>

<td>{4273EDBD-AC1D-40d3-9FB2-095C621B552D}</td>

</tr>

<tr>

<td>Multiple Lines of Text</td>

<td>{E0DECE4B-6FC8-4a8f-A065-082708572369}</td>

</tr>

<tr>

<td>Option Set</td>

<td>{3EF39988-22BB-4f0b-BBBE-64B5A3748AEE}</td>

</tr>

<tr>

<td>MultiSelect Option Set</td>

<td>{4AA28AB7-9C13-4F57-A73D-AD894D048B5F}</td>

</tr>

<tr>

<td>Two Options</td>

<td>{67FAC785-CD58-4f9f-ABB3-4B7DDC6ED5ED}</td>

</tr>

<tr>

<td>Status</td>

<td>{3EF39988-22BB-4f0b-BBBE-64B5A3748AEE}</td>

</tr>

<tr>

<td>Status Reason</td>

<td>{5D68B988-0661-4db2-BC3E-17598AD3BE6C}</td>

</tr>

<tr>

<td>Whole Number</td>

<td>{C6D124CA-7EDA-4a60-AEA9-7FB8D318B68F}</td>

</tr>

<tr>

<td>Floating Point Number</td>

<td>{0D2C745A-E5A8-4c8f-BA63-C6D3BB604660}</td>

</tr>

<tr>

<td>Decimal Number</td>

<td>{C3EFE0C3-0EC6-42be-8349-CBD9079DFD8E}</td>

</tr>

<tr>

<td>Currency</td>

<td>{533B9E00-756B-4312-95A0-DC888637AC78}</td>

</tr>

<tr>

<td>Date and Time</td>

<td>{5B773807-9FB2-42db-97C3-7A91EFF8ADFF}</td>

</tr>

<tr>

<td>Lookup</td>

<td>{270BD3DB-D9AF-4782-9025-509E298DEC0A}</td>

</tr>

<tr>

<td>Customer</td>

<td>{270BD3DB-D9AF-4782-9025-509E298DEC0A}</td>

</tr>

<tr>

<td>Owner</td>

<td>{270BD3DB-D9AF-4782-9025-509E298DEC0A}</td>

</tr>

<tr>

<td>Unique ID</td>

<td>{5546E6CD-394C-4bee-94A8-4425E17EF6C6}</td>

</tr>

</tbody>

</table>

My custom dialog now looks like this. If you don't know how to create the buttons, follow the instructions in Bob's blog post.

<div class="wp-block-image">

<figure class="aligncenter">![](/uploads/2019/06/Close-My-Opportunity.png)

<figcaption>You can see that I also added a description field, it is just a simple multiple lines of text field.</figcaption>

</figure>

</div>

The dialog consists of only two fields - _Status Reason_ and _Description_. I did want to include the _Actual Revenue_ field, however it seemed impossible to use, because the currency data type is not supported in custom dialogs (same for the _Competitor_ field which is a lookup data type). I didn't include the _Close Date_ field, we can just use the TS function _Date()_.

Now there comes the interesting part. The system dialog box closes the opportunity in the chosen status reason and refreshes the page. How are we going to do this? Simply, we are going to write a TypeScript library which will contain all the functions we need. If you wonder why I'm using TypeScript, it's because of the async requests. I recommend you to use it, because you never know when you are going to need to do something asynchronous.

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/3dc8de50375696ffedf9f872d9e79b5f#file-CustomDialogLibrary-ts">View this gist on GitHub</a></noscript>

</div>

Those are all the functions we need. The first one opens the dialog and **is being called from our ribbon button**. Then, there is a function for the Cancel button and the OK button.

You can see the _my_guid_ attribute in the _inputParameters_ object. I've added this field as query string input parameter. You can send to your dialog anything you need from the form context.

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/3dc8de50375696ffedf9f872d9e79b5f#file-FormParameters-xml">View this gist on GitHub</a></noscript>

</div>

Now that we have all the input that we need for the _loseOpportunity_ request, we can write the function that closes the opportunity.

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/3dc8de50375696ffedf9f872d9e79b5f#file-CloseOpportunity-ts">View this gist on GitHub</a></noscript>

</div>

And that's it! We have our fully working custom dialog. In order to use it, you have to create a button in the _Ribbon Workbench_ and include it on a form in your solution. You can download the solution here.

<div class="wp-block-file aligncenter">[Download](/uploads/2019/06/CustomDialog.zip)</div>