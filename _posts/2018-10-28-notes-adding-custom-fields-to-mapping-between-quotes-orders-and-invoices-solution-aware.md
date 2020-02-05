---
author: tomas.prokop
title: >-
  Notes: Adding custom fields to entity maps between Quotes, Orders and Invoices
  (solution aware)
slug: >-
  notes-adding-custom-fields-to-mapping-between-quotes-orders-and-invoices-solution-aware
id: 265
date: '2018-10-28 13:17:58'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - CDS
  - Hack
  - Internal
  - Mapping
  - Plugin
  - Solution
---

Finally, it's the weekend and I have some time to focus on an issue which bothered our team for a few months. As always we wanted to do it the right way so it will be fast, reusable, continuous integration compatible and without spawning unnecessary workflows and plugin instances. ![](/uploads/2018/11/POWERPNT_2018-11-03_14-37-46-e1541252402156.png) 

# The Problem

Sales solution makes it easy to go through the lead to cash process. One of the out-of-box helpers is copying data between entities. Example:

1.  Add products to opportunity
2.  Create a quote by clicking on a little plus sign in the top right corner of product list. The quote gets relevant fields prepopulated using data from opportunity including products lines
3.  Edit quote and create an order from it using a ribbon button.
4.  Order also gets all the data from the quote.
5.  And the same goes for invoices.

This works well until we want to customize this behavior or do it for other entities. Unfortunately CRM server (the platform side) still has a lot of hidden hacks. Now with Common Data Model we want to reuse entities and processes wherever possible.

# Use Cases

There are few legitimate reasons for adding custom fields to sales line entities. We have these:

*   Customer Facing Product Name
    *   Specify different text for product names in the actual quotes and invoices.
*   Date of delivery
    *   Have delivery dates specified for each line item.

# Background

As you may already know you can map some fields which have 1:N relationship. [https://docs.microsoft.com/en-us/powerapps/maker/common-data-service/map-entity-fields](https://docs.microsoft.com/en-us/powerapps/maker/common-data-service/map-entity-fields) Let's have a look how we use this in code. Once you have maps in place you can create a new record with InitializeFrom and it will have values prepopulated according to the map. [https://docs.microsoft.com/en-us/dynamics365/customer-engagement/web-api/initializefrom](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/web-api/initializefrom) Here is a sample how it works:

1.  Create the **InitializeFromRequest**
2.  Set **Contact** as the Target
3.  Set
4.  the reference of parent **Account** as the **EntityMoniker** which you want to copy the data from
5.  Execute the **InitializeFromRequest**
6.  Read the object with copied data from Account from **InitializeFromResponse**
7.  Make other modifications
8.  Create the record

<pre class="lang:c# decode:true">InitializeFromRequest request = new InitializeFromRequest();

request.TargetEntityName = "contact";

request.EntityMoniker = new EntityReference("account", idOfAccount);

InitializeFromResponse response = (InitializeFromResponse)organizationService.Execute(request);

if (response.Entity != null)
{
Entity newContactRecord = response.Entity;

newContactRecord.Attributes.Add("firstname", "Tomas");

organizationService.Create(newContactRecord);
}
</pre>

  The internal Sales solution uses this too. When you hit Create Order button they do this. But you don't see their EntityMaps in solution editor because there is no direct relationship between mapped entities. Field mappings aren’t actually defined within the entity relationships, but they are exposed in the relationship user interface. ![](/uploads/2018/11/POWERPNT_2018-11-03_14-29-42.png)

# Solution

So can't edit mappings for these entities using UI editor for relationships because there is no direct relationship between the records but the entity map exists in the database (entitymapbase table). When you google a bit you'll find a way to get an ID of the entity mapping table using an organization service endpoint:

> /XRMServices/2011/OrganizationData.svc/EntityMapSet

  (This can be done with Web API endpoint too: **/api/data/v9.0/entitymaps?$filter=contains(sourceentityname,%27quotedetail%27)**) If you try to list this EntityMapSet resource, you probably won't find it because it's hidden in some instances (I've seen both cases). But you can filter mapping tables by specifying source and destination in a request query and it works:

> /XRMServices/2011/OrganizationData.svc/EntityMapSet?$select=EntityMapId&$filter=SourceEntityName%20eq%20%27<span style="color: #ff0000;">**salesorderdetail**</span>%27%20and%20TargetEntityName%20eq%20%27<span style="color: #ff0000;">**invoicedetail**</span>%27

<pre class="lang:default highlight:0 decode:true"><?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xml:base="https://thenetworg.crm4.dynamics.com/XRMServices/2011/OrganizationData.svc/">
   <title type="text">EntityMapSet</title>
   <id>https://ABC.crm4.dynamics.com/XRMServices/2011/OrganizationData.svc/EntityMapSet</id>
   <updated>2018-08-25T15:32:50Z</updated>
   <link rel="self" title="EntityMapSet" href="EntityMapSet" />
   <entry>
      <id>https://ABC.crm4.dynamics.com/XRMServices/2011/OrganizationData.svc/EntityMapSet(guid'6825c078-5459-e811-a84a-000d3ab6b1ed')</id>
      <title type="text" />
      <updated>2018-08-25T15:32:50Z</updated>
      <author>
         <name />
      </author>
      <link rel="edit" title="EntityMap" href="EntityMapSet(guid'6825c078-5459-e811-a84a-000d3ab6b1ed')" />
      <category term="Microsoft.Crm.Sdk.Data.Services.EntityMap" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme" />
      <content type="application/xml">
         <m:properties>
            <d:EntityMapId m:type="Edm.Guid">6825c078-5459-e811-a84a-000d3ab6b1ed</d:EntityMapId>
         </m:properties>
      </content>
   </entry>
</feed>

</pre>

Look for the EntityMapId element and find the map ID. It is different in every environment since the metadata definition does not contain any identifier. In my case it is **6825c078-5459-e811-a84a-000d3ab6b1ed.** Now I need to open a designer directly using this URL with my Guid:

> /tools/systemcustomization/relationships/mappings/mappingList.aspx?mappingId=%7b<span style="color: #ff0000;">**6825c078-5459-e811-a84a-000d3ab6b1ed**</span>%7d

Here you can add your desired mapping using New button: ![](/uploads/2018/08/chrome_2018-08-25_22-03-15.png) ![](/uploads/2018/08/chrome_2018-08-25_22-04-58.png)

# That's not everything.. Include it in a solution!

<span style="text-decoration: underline;"><span style="color: #ff0000;">**You should never ever ever make unmanaged modifications to the Default Solution in your production environment!!**</span></span> Yeah but you have another problem because the EntityMap does not get exported with your solution in your Dev environment and therefore you are not able to move it to production. There is just no way to include it using UI.

## Import - the easy part

If you are able to construct the mapping during your build process or you have your solutions in source control, you can inject it to customizations.xml right away. 😊 ![](/uploads/2018/08/Code_2018-08-25_22-35-38.png)

<div>

<pre class="lang:default highlight:0 decode:true"><?xml version="1.0" encoding="UTF-8"?>
<EntityMaps>
   <EntityMap>
      <EntitySource>quotedetail</EntitySource>
      <EntityTarget>salesorderdetail</EntityTarget>
      <AttributeMaps>
         <AttributeMap>
            <AttributeSource>tntg_productcustomername</AttributeSource>
            <AttributeTarget>tntg_productcustomername</AttributeTarget>
         </AttributeMap>
      </AttributeMaps>
   </EntityMap>
</EntityMaps></pre>

After importing this CRM server will append the Attribute map to the existing EntityMap and you are good to go: ![](/uploads/2018/08/chrome_2018-08-25_22-33-57.png) Great!

## Export - where it gets dirty

Take this part more as notes from my research than a guide. After the initial euphoria immediately came disappointment. I tried to export the solution to check whether it contains the necessary mapping information. It did not. ![](/uploads/2018/08/Code_2018-08-26_02-21-13.png) Out of desperation I've started searching the solution export logic in assemblies from Dynamics CRM installation.</div>

Here is the code responsible for determining whether an EntityMap gets exported.

<pre class="lang:c# decode:true">private bool DoesEntityMapNeedToBeSkipped(Hashtable entitiesTable, EntityMappingStruct mapping)
{
    if (entitiesTable != null)
    {
        if (!entitiesTable.Contains(mapping.EntitySource.EntityInfo.LogicalName) && !entitiesTable.Contains(mapping.EntityTarget.EntityInfo.LogicalName))
        {
            return true;
        }
        if (mapping.EntitySource.IsCustomEntity && !entitiesTable.Contains(mapping.EntitySource.EntityInfo.LogicalName))
        {
            return true;
        }
        if (mapping.EntityTarget.IsCustomEntity && !entitiesTable.Contains(mapping.EntityTarget.EntityInfo.LogicalName))
        {
            return true;
        }
    }
    return false;
}</pre>

This indicates that the map will be skipped if:

*   Both Target and Source entities are not included in the solution
*   One of the entities is custom and is not included in the solution

I have also found this: [https://stackoverflow.com/a/41273138](https://stackoverflow.com/a/41273138)

> I have also seen where you have to have both the relationship, and both fields defined the mapping in the solution in order for the mappings to be exported... So if I have Entity A that has a Mapping to B, for fields A.1 to B.1 and A.2 to B.2, I have to make sure that the relationship, and fields A.1, A.2, B.1 and B.2 have been added to the solution as well, or else they don't get exported. After some further testing, in order for Lookup Attributes to be included in the Export of a Mapping, the Target Attribute field **MUST BE** included in the solution!

So I guess maps with both out-of-box entities are automatically skipped.

<div>BTW this is how individual attribute map looks like:</div>

> /api/data/v9.0/attributemaps?$filter=_entitymapid_value%20eq%2013180255-43fd-e711-80f9-00155d036800

![](/uploads/2018/08/chrome_2018-08-25_23-45-01.png)