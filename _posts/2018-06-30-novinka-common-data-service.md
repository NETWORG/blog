---
author: Tomas Prokop
title: Novinka Common Data Service 2.0 v souvislostech
slug: novinka-common-data-service
id: 144
date: '2018-06-30 21:19:41'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - Common Data Service
  - Dynamics 365
  - Dynamics CRM
  - licence
  - Sales Professional
---

Pokud to trochu sledujete, ale začínáte se ztrácet v posledních novinkách v oblasti informačních systému Microsoftu, tak určitě nejste sami. Microsoft totiž chodí s kladivem, rozbíjí je na menší části a se snaží poskládat moderní platformu, která obstojí v dnešním cloudovém světě, kde se vše mění dříve než se to pořádně dostane k uživatelům. Níže si můžete přečíst o situaci v kontextu CRM a platformy Common Data Service, kterou považuji za zásadní pokrok. Přinese totiž robustní informační systémy i do menších organizací a prováže technologie, které si dodnes spolu povídaly jen draze a složitě. V úvodu ještě upozorním, že vše zde napsané ohledně budoucnosti je pouze mojí osobní interpretací informací, které se postupně uvolňují. Doporučuji začít u historie Dynamics CRM: https://blog.networg.com/2018/05/20/historie-dynamics-crm-ce-365-for-sales/

# Vše je zase jinak a bude to jen horší 😊

Důležité je si uvědomit, že doby, kdy jsme dostali jednu verzi systému, okolo té jsme se zafixovali a vydrželi několik let, jsou nenávratně pryč. Po změnách architektury, které umožní hyperscale provoz v cloudu, kterými si procházel (a prochází) Exchange a SharePoint, toto čeká i Dynamics. Již dnes Microsoft vydává aktualizace rychleji, než je stíhá dokumentovat: [https://support.microsoft.com/en-us/help/2925359/microsoft-dynamics-crm-online-releases](https://support.microsoft.com/en-us/help/2925359/microsoft-dynamics-crm-online-releases) Jako Microsoft partner už nikdy nebudete v obraze a jako zákazníci jste naprosto bez naděje. Vše, co dnes vymyslíte a naimplementujete, bude za pár týdnů zastaralé. Dobrá zpráva je, že tyto změny většinou řeší naše dlouhodobé problémy. Přináší skvělé nové možnosti.

# Proč?

Vše má samozřejme své opodstatnění a osobně mi to, přestože jsem z toho jako všichni vystrašený, dává smysl. Celé to směřuje ke stavu, kdy se aplikace stanou mnohem jednodušší na údržbu, IT projekty budou levnější, zákazníci budou mít více možností uchopit konfigurace samostatně a budeme mít možnost tyto technologie dostat i do těch nejmenších organizací. Bude to trvat, než si na to zvykneme, bude to doprovázet mnoho potíží i zničených business modelů, ale jsem přesvědčený, že stojí to za to. O celém konceptu Citizenship Developer budu psát v přístích článcích.

# Common Data Service

Pojďme si tedy představit tu novinku. Myšlenka je jednoduchá - máte jednu spolehlivou databázi pro veškerá firemní data, která komunikuje se všemi ostatnímy systémy, které využíváte. Nemusíte řešit její provoz, zálohy, redundanci, konfiguraci serveru, CALy. Přístup k datům nastavujete na úrovni entit a platí pro všechny aplikace, přes které s nimi uživatel manipuluje. Aplikaci pro uživatele vytvoříte podobným stylem jako prezentaci v PowerPointu a business logiku řídíte opět na jednom místě a co nejblíže k datům. Mnoho scénářů pro vás předpřipravil Microsoft v podobě Dynamics 365. Common Data Service je solidní základnou pro řízení firemních dat a budování veškerých interních i externích agend. Do hloubky se budeme platformou zabývat později. Zjednodušeně se jedná o následující:

*   Business aplikace
*   Analytické nástroje
*   Integrace systémů
*   Jedna centrální spravovaná databáze
*   Jednotný datový model, který lze rozšířit
*   Business logika na jednom místě
*   Platforma pro vývoj

![](/uploads/2018/06/POWERPNT_2018-06-30_21-07-34.png)  

# Co to znamená pro Dynamics?

Pokud jsem psal o změnách a rychlosti, tak se to nejvíce týká těch, kteří se snaží zorientovat a nastavili své IT vize okolo produktů Dynamics. Dynamics je produktový název a kategorie, podobně jako třeba Office. To zůstane i nadále. Novinka je oddělení platformy, která bude zřejmě dále již známa pod označením PowerApps. Na začátku dubna 2018 byl představený refresh nástroje Common Data Service - CDS 2.0, který do té doby potichu bez větší pozornosti vznikal v pozadí pod střechou PowerApps. Tato technologie bude dle mého názoru naprosto zásadní pro další rozvoj informačních systémů postavených nad řešeními Microsoftu. Tato velká aktualizace CDS v podstatě zahodila vše, co do té doby vzniklo pod tímto názvem a Microsoft začal migrovat PowerApps na novou platformu - refaktorovaný CRM server, který musel projít zásadní přestavbou. Bylo nutné oddělit agendy CRM od platformy a rozbít závislosti platformy na komponentách na ní postavených. Nově máme tři typy aplikací PowerApps:

*   **Canvas** - jednodušší aplikace, které se vytvářejí podobně, jako PowerPointová prezentace
*   **Model Driven** - Dynamics (CRM) App Modules s Unified User Interface a Custom Controls Framework
*   **Web Client** -Současné Dynamics 365 Customer Engagement webové prostředí

![](/uploads/2018/05/cds.jpg)

# Instance a licence

Zatím to zní skvěle, že? Zchlazení přijde v posledním bodě, kdy začnete zkoumat cenu. Aktuálně totiž vše nejvíce brzdí licenční model, který není ještě úplně vyjasněný. Je tu skvělá platforma, jejíž použití brání nedotažený způsob toho, jak ji zakoupit. Je to komplikované tím, že se spojily týmy a modely, které byly dříve tvrdě oddělené. Máme tedy aktuálně k dispozici následující licence:

*   PowerApps P1
*   PowerApps P2
*   Dynamics 365 for Team Members
*   Dynamics 365 for Sales Professional
*   Dynamics 365 for Sales Enterprise

Zatím jsou instance odděleny pro různá předplatná. To je velká škoda. Aktuální obrázek z Licensing Guide (červen 2017):

# ![](/uploads/2018/06/chrome_2018-06-30_20-25-37.png)

Pokud dnes vytvoříte CDS environment, dostanete další oddělenou instanci. V praxi vznikne potřeba mixovaného prostředí zejména kvůli ceně. I kdyby toto rozdělení zůstalo i v budoucnu, nabízí se využití virtuálních entit, které bez nutnosti kopírování dat instance propojí. Doufejme, že toto bude velmi brzy adresované, protože je to velká škoda. Další informace bychom se mohli dozvědět již 15\. července na partnerské konferenci Inspire. [https://partner.microsoft.com/en-US/inspire](https://partner.microsoft.com/en-US/inspire)

# Už bylo na čase

Zákazníci, partneři i Microsoft již dlouhou dobu pociťují velký problém - rozdělené produkty Dynamics, jejichž agendy se překrývají, si spolu neumí spolehlivě povídat a jsou technologicky z historických důvodů postavené každý jinak. Zde je jeden již velmi neaktuální slide (od té doby Business edition - zrušeno, NAV+AX -> Finance and Operations, Tenerife -> Business Central, Retail, ...), který dokonale ukazuje, kolik problémů to musí Microsoftu způsobovat při údržbě: ![](/uploads/2018/06/portfolio-roadmap.jpg) V minulosti již proběhlo několik neúspěšných pokusů o sjednocení a nejznámějším z nich je Project Green. CDS for Apps tedy stojí na základech XRM (CRM server). Microsoft se setejně tak mohl rozhodout pro pokračování s technologií pod Dynamics AX, která na tom byla o něco dále než CRM třeba z pohledu hyper scale databáze. Rozhodnutí padlo na CRM. V blízké budoucnosti se toho možná dočkáme s Common Data Service.