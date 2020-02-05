---
author: Frantisek Capek
title: Dynamics 365 - Subgrids without relationship
slug: dynamics-365-subgrids-without-relationship
id: 808
date: '2019-07-22 10:59:38'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - CDS
  - Customer Engagement
  - Dynamics 365
  - FetchXML
  - subgrid
---

Have you ever wanted to display a view from different entity and there was no straight relationship? Worry not! I finally figured out how to do it. So, let's just jump into it!

Prerequisites:  

1) Have two entities with common related entity, my ex. Opportunity, Tag, related entity Account (there is a N:N relationship between Tag and Account).  

2) Have a subgrid on entity (Opportunity) which shows all the records from the entity you want to show (Tag) and get it's FetchXML using the advanced find.  

3) If you don't know the name of the intersect entity, find it using the REST Builder - it should look like this: prefix_entity1_entity2\. [https://github.com/jlattimer/CRMRESTBuilder](https://github.com/jlattimer/CRMRESTBuilder)

If you meet all the requirements, all you need to do is to write a custom TypeScript code (compiled to JavaScript) which will "filter" the subgrid - the function is called setFilterXml, but it completely replaces the FetchXML.  
_Note that we are creating a new FetchXML, it should look same as the one from the subgrid except the conditions with filter type "or"._

It should look like this:

<div class="wp-block-coblocks-gist">

<noscript><a href="https://gist.github.com/MaceWindu1/43f6de12837e35f6fcbc7d2168969f68#file-filterSubgrid-ts">View this gist on GitHub</a></noscript>

</div>

And that's it!  
In order to use this function, add it onload of the form and you're good to go.