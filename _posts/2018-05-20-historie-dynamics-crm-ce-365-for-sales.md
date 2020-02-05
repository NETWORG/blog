---
author: tomas.prokop
title: Historie Dynamics CRM / CE / 365 for Sales
slug: historie-dynamics-crm-ce-365-for-sales
id: 133
date: '2018-05-20 15:57:40'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - Common Data Service
  - CRM
  - Dynamics
  - PowerApps
---

Jako základ pro další články a informace o platformě považuji za nezbytné, abych přiblížil historický kontext a vývoj platformy, která dnes stojí za službou Common Data Service a jistě ji čeká další velká budoucnost, protože byla zvolena jako základ pro budoucí business aplikace Microsoftu a stejných způsobem je otevřená jako stavební základ pro nás a různé aplikace třetích stran. Podívejme se tedy na hlavní milníky jedné z větví Dynamics - CRM. Vybral jsem ty nejzajímavěší novinky v každém hlavním vydání, které se nějak vztahují k architektuře platformy.

# Začátky

Nyní je to již 15 let od uvolnění první verze v roce 2003\. Historie tohoto systému ale sahá až k roku 2000 a produktu iCommunicate.net, který sloužil pro správu zákazníků a základní service management. Microsoft tento produkt koupil v roce 2001 a společně s 10 zaměstnanci, kteří přešli pod Microsoft začal vývoj webového CRM. V úvodu to měla být pouze ukázková aplikace, která měla demonstrovat možnosti platformy, na které je postavená.

# 2003: CRM 1.0-1.2

![](/uploads/2018/05/crm1.jpg) V roce 2003 pod názvem Business Solutions Customer Relationship Management 1.0 Microsoft vypustil první verzi, která zatím neumožňovala příliš customizací, ani například vytváření vlastních entit. Jednou z hlavních výhod byla možnost propojení Outlookem, který byl již v té době velmi rozšířený. Dokonce mohli uživatelé využít offline synchronizace.

# 2005: CRM 3.0 Danube Phase II

![](/uploads/2018/05/crm3.jpg) Verze 2 byla po několika odloženích přeskočena a v roce 2005 byla uvolněna verze 3\. Poprvé jsme se setkali s označením Dynamics. V mezičase se začaly objevovat informace o projektu Green, který se poprvé snažil spojit všechny business aplikace Microsoftu pod jednu platformu (AX, CRM, AX, GP, SL). Nicméně až v roce 2007 byl tento projekt zrušen a produkty šly dále svojí vlastní cestou. Ve verzi 3 začalo uživateské rozhraní, které imitovalo Outlook.

# 2007: CRM 4.0 Kilimanjaro/Titan (4.0.7333)

![](/uploads/2018/05/lokup_3.png) V roce 2007 byla uvolněna verze 4.0:

*   Vylepšený model zabezpečení
*   Importy Mail-merge
*   Více než 25 jazyků a podpora více měn
*   Reporting wizard pro bežné uživatele
*   Zobrazení se sloupci z příbuzných entit
*   Automatizace pomocí Workflow Foundation
*   Propojení s Microsoft Office Communications Server 2007 (později Lync/Skype for Business)
*   Ukazatelé stavu kolegů (Online, Pryč, ...), které se dnes vrací v Unified Interface bez závislosti na lokální instalaci Skype for Business a použití Internet Exploreru.

Tato verze umožňovala provozovat více organizací na jednom serveru (multi-tenant). Poprvé bylo možné využít hostovanou verzi CRM Online.

# 2010: CRM 2011 Polaris (5.0.9688 - 5.0.9690)

![](/uploads/2018/05/crm2011.png) Tato verze přinesla z dnešního pohledu velmi podstatný koncept řešení (solutions) a vrstvení úprav. Ten umožňuje partnerům importovat do systému spravované úpravy, které je možné přidávat a odebírat podobně, jako aplikace v telefonu. Nově se objevily i řídící panely, které přinesly vizuální změnu a možnosti graficky reprezentovat data v reálném čase. Byl přidán i ribbon v dobovém stylu z Microsoft Office. Pro extenzibilitu přibyly OData a WCF endpointy. V Online verzi nová možnost sandboxovaných pluginů. Entity již mohly mít více formulářů. **Aktualizace byly v té době naplánované dvakrát ročně. Původně byly běžné 2-3 leté cykly.**

# 2013: CRM 2013 Orion/Leo (6.0.0 - 6.1.5)

![](/uploads/2018/05/crm2013.png) Zásadní redesign, nové uživatelské rozhraní MoCA, které je již dnes nahrazeno Unified Interface. Hlavním tématem této verze byla mobilita. Přibyly klientské aplikace pro tablety a mobilní telefony (Windows Phone, iPhone, Android a Windows 8). Nově bylo možné pro formuláře definovat Business Rules místo toho psát JavaScript pro každou jednodušší klientskou logiku.

*   Business Process Flows (toky)
*   Business Rules - XAML definice pravidel, které se na straně klienta překládají do JavaScriptu
*   Poznámky a přílohy pro všechny záznamy
*   Automatické ukládání

# 2014: CRM 2015 Vega/Carina (7.0.0 - 7.1.2)

![](/uploads/2018/05/crm2015.jpg)

*   Velmi praktickou novinkou byly
    *   Calculated Fields - pole, která jsou jako fukce při dotazu automaticky počítána
    *   Rollup fields - pole, která jsou periodicky na pozadí počítána z hodnot příbuzných entit
*   Business Rules lze spouštět i na straně serveru.
*   API pro Business Process Flows
*   Doporučení produktů - cross selling
*   Offline drafty v mobilních aplikacích
*   Výrazně vylepšené vyhledávání (rychlý fulltext)
*   Field Level Security - oprávnění ana úroveň polí a šifrování
*   Nested Quick Create - zanořené dialogy pro rychlé vytvoření (vytvoření několika příbuzných záznamů naráz)

# 2015: CRM 2016/Dynamics 365 Ara/Naos/Centaurus (8.0.0-8.2.2)

*   Voice of the Customer, Interactive Service Hub, FieldOne (později Field Service), Portals (dřívější ADX Studio)
*   Volání Business Rule z Business Process Flow
*   Vlastní Word a Excel dokumentové šablony nad každou entitou
*   Nová odlehčená Outlook App, která funguje na webu i v telefonech
*   Integrace s Excelem - zpětná aktualizace záznamů v CRM
*   Nové kontrolky - např. editovatelné zobrazení
*   Server side synchronizace emailů i v hybridním scénáři
*   **Relevance Search = **kvalitnější full-textové vyhledávání pomocí externí služby Azure Search

![](/uploads/2018/05/relevance-search.png)

*   **Offline synchronizace** pro mobilní (i desktopovou) aplikaci = synchronizavané záznamy na pozadí, offline přístup k datům
*   App modules - nový princip rozdělení systému na menší aplikace (komponenty, oprávnění)

  ![](/uploads/2018/05/mobileoffline.png)   Následující verze podrobně popíši v dalších článcích.

# 2017: Dynamics 365/CDS Potassium (9.0.0 - 9.0.2)

![](/uploads/2018/05/chrome_2018-05-30_20-35-21.png) ![](/uploads/2018/05/chrome_2018-05-30_20-36-53.png)

*   Separace aplikací a platformy. Roztrhání závislostí.
*   **Unified User Interface** (pouze nad App modules) - nové uživatelské rozhraní, které je responzivní a konzistentní napříč zařízeními
*   **Refreshed Web Interface - **vylepšený webový klient = kondenzovanější rozhraní, vylepšené ovládací prvky, ohraničení elementů
*   Multi Select Option Sets - možnost vybrat více hodnot z číselníku zároveň podporovanou cestou
*   Integrace Microsoft Flow
*   Virtuální entity - integrace různých datových zdrojů bez nutnosti kopírovat data

# ?: Dynamics 365/CDS Calcium (9.1?)

Čekáme <span class="emoji">😊</span><span class="emoji">.</span>