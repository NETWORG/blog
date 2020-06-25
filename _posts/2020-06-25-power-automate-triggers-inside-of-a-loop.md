---
author: Matěj Samler
title: Power Automate triggers inside of a loop
date: '2020-06-25T18:20:00+0200'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - Microsoft Flow
  - Power Automate
---

# Power Automate triggers inside of a loop

As we all probably know, every Flow in Power Automate begins with a trigger and continues with multiple actions, branches and conditions. But have you ever considered putting a trigger in the middle of a flow? 

That's right. It is entirely possible to put another trigger inside of an action. We can use this technique to create an asynchronous pattern inside of a single flow. Consider the following example:

You receive an email with an attachment, which you then want to upload to a document management system and reply to the email with a URL link to that uploaded file. This all sounds nice and easy, but the creation of the file in DMS does not necessarily need to be synchronous, or the connector we are using can just tell us that the action itself was successful, without giving us information about the result, such as the URL we want to return.

We can go about it with this pattern:

![](/uploads/2020/06/asyncFlow.jpg)

As you can see, we created two separate branches. The branch on the right just creates a file and finishes execution. The one on the left on the other hand begins with a trigger, and that will behave as a separate mini flow, triggering with the trigger and sending the email only after the file itself was created. This pattern is pretty handy if you need to wait for something to finish and you do not necessarily need it to be its own separate flow. 

Now that we know that you can create a trigger basically anywhere...what is stopping us from creating it inside of a loop? 

I know what you are thinking: "Why would we ever need to have triggers inside of a loop? That's madness! That can never work"
I get why you might be skeptical, but I assure you that it works, and I can give you a pretty simple use case of this technique, that you have probably considered before: 

 **Dynamic Parameters inside of a trigger.** 

Let me explain. Imagine you want to create a flow that reads messages from a [service bus](https://azure.microsoft.com/en-us/services/service-bus/) using Power Automate. What you will discover after selecting the trigger is that you need to hardcode the topic and subscription names inside of the trigger for it to work, because there is no way to give Power Automate some run properties. This would be no problem, but what if you want to listen to two subscriptions? Three? Fifty? 

![](/uploads/2020/06/trigger.jpg)

The obvious solution would be creating multiple flows, but that can become annoying really quick, not to mention it is not really supportable. One thing we can do is provide properties to the trigger from external source, using the technique we talked about earlier. Let's imagine we have a configuration record in [CDS](https://powerapps.microsoft.com/en-us/common-data-service/) that keeps this information, and once we create it we want to start listening. 

![](/uploads/2020/06/triggerwithconfig.jpg)

Great, now when we create config we can listen. But this will only happen once, and once the first message arrives and is handled, we would have to create the record again, which does not help us much. What if instead, we get all these configuration records, wait for a while to see if some new message arrived and then launch it all again. 

![](/uploads/2020/06/finalflow.jpg)

This is the general idea. Basically, every X minutes, we generate a dynamic number of mini flows, number of depends on the number of config records, we can handle for them to trigger and then recur again. Of course, since we are still inside Power Automate, we can still make parallel branches, loops and maybe even additional triggers inside of the loop. the possibilities are endless.

If you are going this way, there are a few things I recommend you do: 
- Set timeout on the triggers
  - If you are recurring, you need to make sure that the flow runs only once, otherwise there might be problems. To do this, you can set timeout for the mini flow triggers using [this format](https://www.digi.com/resources/documentation/digidocs/90001437-13/reference/r_iso_8601_duration_format.htm)
 - Return success
   - Timeout is handled as a fail in the eyes of Power Automate. I recommend adding a terminate step after the flow, which will return success if the flow succeeds or times out. You can do that by using runAfter and it will make sure that errors are still handled properly.

  ![](/uploads/2020/06/timeout.jpg)

 - Do it in bulk
   - In the example above, I am waiting for a single message. I recommend waiting for more, since once the mini flow fires, it has to wait until the entire recurrence happens. Configure your bulk operation as such so that once every X minutes, you download everything that happened since.

So in summary: If you need to pass some properties into a Power Automate Flow trigger, and it would require you to have many different flows with many different values, you can circumvent this by adding triggers into a for loop. 


