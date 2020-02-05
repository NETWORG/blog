---
author: Tomas Prokop
title: >-
  Co pro nás znamená nový způsob doručení aktualizací Dynamics 365 CE a jak se
  připravit
slug: >-
  co-pro-nas-znamena-novy-zpusob-doruceni-aktualizaci-dynamics-365-ce-a-jak-se-pripravit
id: 287
date: '2018-08-26 18:29:50'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - aktualizace
  - customer driven update
  - Customer Engagement
  - Dynamics 365
  - Dynamics CRM
  - First Release
  - Monitoring
  - update
  - Verzování
  - w
---

Microsoft chce dostat všechny zákazníky v cloudu na poslední (a tu nejlepší) verzi, a tak mění kompletně styl vývoje, podpory a nasazovaní aktualizací na kontinuální model s největší možnou kadencí. Od V9 již není možné plánovat, kdy bude instance aktualizována. Od února 2019 nejspíše všichni v online prostředí poběží na stejné verzi. Již nebude možné zůstávat pozadu. Krátce na to přijde nová verze V10 (duben 2019). Nové funkcionality a případné breaking changes budou vydávány dvakrát ročně v dubnu a listopadu. Microsoft se pokusí zajistit zpětnou kompatibilitu pro rozšíření a úpravy.![](/uploads/2018/08/chrome_2018-08-26_17-18-42-e1535298413472.png)  Aktuálně jsme v online prostředí naučeni využívat možnost plánovat, která již nebude od V9 k dispozici. ![](/uploads/2018/08/CDU_schedule_your_update.png)

# Benefity a problémy

## Partneři

Budou muset podporovat pouze jednu verzi. To znamená, že pokud má partner správně nastavený proces pro solution development, tak tímto výrazně úspoří čas, protože nebude muset držet mnoho větví vývoje pro různé verze (ale pozor, pořád tu máme on-premises zákazníky). Dále ale bude nutné, aby se pečlivě připravili na přicházenící změny, které Microsoft bude oznamovat několik měsíců dopředu pomocí release notes. ![](/uploads/2018/08/chrome_2018-08-26_17-17-50.png) ![](/uploads/2018/08/chrome_2018-08-26_15-45-40.png) Osobně ale v tomto vnímám velké riziko v hotfixech a minor aktualizací, které jsou nasazovány průběžně. Již několikrát se nám stalo, že v rámci minor patchů, které jsou průběžně téměř každých 14 dnů nasazovány všem zákazníkům na V9, došlo k porušení funkcionality bez našeho vědomí. Nejčerstvější případ je například to, že jednoho rána se nám ve všech vygenerovaných PDF dokumentech začal zobrazovat nějaký interní log, který vypisoval ID využitých záznamů. Očividně platformní tým mění generování šablon je to již druhý problém v této SDK message za poslední tři měsíce, který jsme byli nuceni nezávisle na sobě eskalovat. ![](/uploads/2018/08/OUTLOOK_2018-08-26_17-59-12.png) Musím ale vyzdvihnout proces, který má aktuálně Microsoft nastavený pro opravy chyb. Stačí založit service request na https://admin.dynamics.com a dobře popsat problém. Opravu je Microsoft schopný díky novému způsobu aktualizací a vývoje nasadit opravdu rychle. Minulý týden jsme například řešili, plnou databázi díky logům z workflows při importu. Nebylo možné je hromadně smazat a bez přístupu k databázi jsme neměli možnost nic dělat. Ještě ve stejný den byl problém vyřešen. Velmi se těšíme na to, že Microsoft plánuje zpřístupnit Issues z jejich interního backlogu pro partnery pod NDA. To znamená, že řešení problémů bude ještě rychlejší a budeme dostávat notifikace při změnách stavu. Doufejme, že již nenarazíme na tak komplikovaný přechod, jako byl V8.2 -> V9\. Tato historicky nejkomplikovanější aktualizace byla doprovázena mnoha problémy. Hlavně díky tomu, že pod kapotou se děly rozsáhly změny ([application / platform separation](/2018/06/30/novinka-common-data-service/)). Díky tomu se vydání tak protáhlo, v době plánovaného vydání vše fungovalo, ale nebylo možné bez újmy přejít ze starších verzí. Naštěstí to Microsoft pozastavil a zalepovalo se to ještě půl roku. Tím se protáhlo zveřejnění dlouho slibované varianty pro small business zákazníky, protože se čekalo na reálné možnosti používat to, na čem běžel douhou dobu Dynamics CRM, jako skutečnou platformu pro různé aplikace i mimo CRM (Power Platform) nad jednou databází. Podobnou, jako má i Salesforce [https://developer.salesforce.com/lightning](https://developer.salesforce.com/lightning).

## Zákazníci

Pro zákazníky to jednoznačně přináší levněší údržbu jejich systému. V případě, že nastane nějaký kritický problém, je pro všechny strany (Microsoft, partnera i zákazníka) jednodušší a rychlejší jej popsat, reprodukovat, řešit a následně nasadit opravu. Pokud se během testování problém neodhalí a přijde se na něj později, má Microsoft mechanismus pro rychlé a flexibilní vydávání hotfixů. V rámci V10 bude možné udělat rollback. Zatím to vypadá, že jej bude možné využít během prvních 24 hodin po aktualizaci. Problémem jsou data a rozhodnutí, jestli je rozumnější přijít o data nebo počkat na hotfix.

# Insider Program a testování

Několik týdnů dopředu bude možné v rámci First Release (podobně jako v Office 365) vyzkoušet v sandboxovém prostředí verzi, která se později nasadí automaticky na produční prostředí. Doufejme, že tímto Microsoft donutí partnery správně nastavit testovací procesy. Ti se naučí správně budou testovat v prostředí, které je obrazem produčního prostředí, ale bez vlivu na uživatele. Zde bude nutné testovat úpravy, integrace a rozšíření třetích stran. Budeme mít přesně definované časové okno pro ladění. Před V10 již bude připraven mechanismus pro First Release: ![](/uploads/2018/08/Upgrades_02b.png) Jak lze vidět na obrázku, major aktualizace se budou nasazovat postupně během několika týdnů. Nejdříve dobrovoníkům, potom menším regionům a nakonec největším zákazníkům.

## Nástroje, monitoring a procesy partnerů

V první řadě je nutné, aby každý partner rozuměl best practises a byl dobře obeznámen s tím, jak funguje solution framework. To je naprostý základ. Velmi často za námi nyní chodí zákazníci od jiných partnerů s mnoha problémy. Jednotící prvek u všech bylo naprosté nepochopení solution developmentu. O těchto příbězích bychom mohli s kolegy povídat hodiny. Pokud je toto zvládnuté, ušetříte si mnoho potíží při upgradech, aktualizacích, přechodech mezi prostředími a migracemi. S Common Data Service se budeme stále častěji setkávat s několika dodavateli u jednoho zákazníka a konečně bude rozbitý ten děsivý vendor lock-in (dodavatel odvede špatnou práci a zákazníkovi se násobí náklady na údržbu a rozvoj systému, čím déle partner prohlubuje technologický deficit), který tak nyní všude okolo vidíme. Dynamics 365 je často v nevýhodě oproti jednodušším CRM systémům především od malých lokálních dodavatelům. Je to zejména kvůli zkušenostem ostatních s dlouhými, nákladnými implementacemi, složitými formuláři a navigací. Na tomto stavu se podílí Microsoft i partneři. Microsoft nebyl zatím schopen připravit startovací balíčky pro různá odvětví a velikosti firem. Mnoho partnerů toto dobře živilo, ale neodbornými zásahy způsobili mnoho škod. Nikdy jsme nenarazili na scénář, který by nějak nešel v Dynamics CRM realizovat. Je to velmi silná platforma, která ale může způsobit noční můry, pokud se s ní nepracuje správně.

> **With great power comes great responsibility.**

Interně se nyní snažíme jít ještě dále a nasazujeme automatické UI testování, které v kombinaci s continuous delivery zajistí jednoduché a rychlé přechody mezi prostředími, automatizovanou detekci problému, rollback našich customizací a eliminaci chyb. [https://github.com/Microsoft/EasyRepro](https://github.com/Microsoft/EasyRepro)

# Průběh aktualizací

Průběžně Microsoft nasazuje hotfixy, které nepřináší nové fuknce. Tyto aktualizace se dále dělí na dvě komponenty:

*   Databáze - Při aktualizacích je nutné spouštět migrační scripty a dělat úpravy ve schématu a datech v databázích. Databáze se verzují, aby bylo možné tento proces dělat transparentně.
*   Aplikace - V cloudu od verze 9 již není nutné držet verzi databáze a serveru stejné. Proto se skupiny serverů, na kterých běží platforma, které obsluhují organizaci, aktualizují zvlášť.

Pro náš service desk jsme si například připravili notifikační mechanismus, který nás upozorňuje pokaždé, když dojde k nasazení nové verze. Díky tomu také ihned víme, když přijde na naši podporu hlášení o nějakém problému, jestli je spojen s aktualizací. ![](/uploads/2018/08/chrome_2018-08-26_16-30-42.png) ![](/uploads/2018/08/Teams_2018-08-26_16-45-36.png)

# Telemetrie

Jedna z věcí, která ve V9 prorostla všemi oblastmi systému, je tememetrie. Nyní je opravdu všude - frontend, backend, SDK messages, CRUD operace. Dynamics začal masivně odesílat anonymní data o tom, jak uživatelé systém využívají. Toto bude využíváno pro licencování (někdo využívá něco, co by neměl) a pro rozhodování o breaking changes. Podle temetrických dat Microsoft pozná, kdy již může odebrat a změnit starší funkcionalitu. Při postupném nasazování aktualizací bude možné ihned identifikovat regresi a včas reagovat. ![](/uploads/2018/08/chrome_2018-08-26_18-30-00.png)

# Onprem

Hodně lidí se v posledním roce ptá a strachuje, jak to bude s on-premises instancemi.

*   Žádné změny v podpoře v průběhu životního cyklu verze
*   Žádné automatické aktualizace
*   Další aktualizace bude uvolněna pravděpodobně do konce tohoto roku

![](/uploads/2018/08/chrome_2018-08-26_15-41-26.png)  

# Jste Microsoft Partner a máte zájem o pomoc?

V TheNetw.org se zaměřujeme na technickou realizaci projektů Power Platform (Dynamics 365). Věříme, že naše technické kapacity nejlépe využijeme pro B2B projekty. Nabízíme konzultace architektury, implementaci projektů, údržbu a rozvoj pro zákazníky našich partnerů jejich jménem. 24.9\. začínáme sérii partnerských školení Business Apps Camp: [https://www.microsoftevents.com/profile/form/index.cfm?PKformID=0x4628382abcd](https://www.microsoftevents.com/profile/form/index.cfm?PKformID=0x4628382abcd)