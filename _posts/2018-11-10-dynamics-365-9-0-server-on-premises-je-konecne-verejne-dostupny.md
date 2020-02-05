---
author: tomas.prokop
title: Dynamics 365 9.0 Server (On-Premises) je konečně veřejně dostupný!
slug: dynamics-365-9-0-server-on-premises-je-konecne-verejne-dostupny
id: 383
date: '2018-11-10 16:52:21'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - '9.0'
  - CDS
  - On-Premises
  - update
---

Přesně před týdnem byl vypuštěn veřejný build verze 9.0.2.3034 ke stažení. Pro On-Premises prostředí je to velká a zásadní aktualizace, která přináší rozdělení aplikací a platformy a funkcionality, které již dlouho známe z cloudové verze Customer Engagement. Velkou a nejviditelnější novinkou je příchod Refreshed a Unified Interface. Klasické rozhraní je vylepšené a uživatelé již neuvidí moře bílé plochy. Získájí přívětivější rozhraní, vylepšené ovládací prvky, ohraničení elementů. Pokud vytvoříte v řešení Application Module, můžete se těšit na nové, krásné a responzivní formuláře (UUI), které jsou konzistentní napříč klienty: ![](/uploads/2018/05/uui.png) Mezi další novinky patří:

*   Multi Select Option Sets – možnost vybrat více hodnot z číselníku zároveň podporovanou cestou. Ale pozor, podpora napříč systémem pořád není 100% (např. nefunkční filtry)
*   Zatímco cloudová verze nabízí inegraci s Microsoft Flow (zřejmě budoucí náhradou za workflows), můžeme si takové propojení zrealizovat i v On-Premises verzi pomocí Webhooks.
*   Virtuální entity – integrace různých datových zdrojů bez nutnosti kopírovat data
*   Vytváření Mobile Task Flows
*   Některé nové designery
*   Relationship assistant

Stažení: [https://www.microsoft.com/en-us/download/details.aspx?id=57478](https://www.microsoft.com/en-us/download/details.aspx?id=57478) Popis funkcí: [https://docs.microsoft.com/en-us/dynamics365/get-started/whats-new/customer-engagement/dynamics365-on-premises-features](https://docs.microsoft.com/en-us/dynamics365/get-started/whats-new/customer-engagement/dynamics365-on-premises-features) Zatím je k dispozici minimum informací, ale průběžně můžete sledovat odkazy: [https://support.microsoft.com/en-us/help/3142345/microsoft-dynamics-365-onpremise-cumulative-updates](https://support.microsoft.com/en-us/help/3142345/microsoft-dynamics-365-onpremise-cumulative-updates) [http://support.microsoft.com/kb/4344184](http://support.microsoft.com/kb/4344184) Český language pack můžete již několik týdnů nalézt zde: [https://www.microsoft.com/cs-CZ/download/details.aspx?id=56970](https://www.microsoft.com/cs-CZ/download/details.aspx?id=56970) Instalační proces podporuje in-place upgrade z některých verzí. Jestli je to možné u vaší verze zjistíte tak, že na spustíte instalačního průvodce, který provede kontrolu. Pokud jste se účastnili private preview nebo máte nainstalovaný insider build, je nutné server nejprve odinstalovat a nainstalovat nový build. Následně v Deployment Manager naimportujete již existující organizace z databáze (z minulé instalace). Při importu dojde k aplikaci migračních scriptů. Pokud plánujete čistou instalaci, při volbě SQL Serveru 2017 můžete narazit na problém u SSRS: https://blog.thenetw.org/2018/03/15/dynamics-365-reporting-extensions-and-sql-server-2017-incompatibility/