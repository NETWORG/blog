---
author: jan.kostejn@thenetw.org
title: Představení Azure Logic Apps
slug: predstaveni-azure-logic-apps
id: 83
date: '2018-04-08 09:53:24'
categories:
  - Azure
  - Czech
tags:
  - Azure Logic Apps
  - Microsoft Flow
---

Služba Azure Logic Apps zjednodušuje vytváření automatizovaných škálovatelných workflows (pracovních postupů), které integrují aplikace a data napříč cloudovými službami a on-premise systémy. K vytvoření vašeho řešení si vyberete z neustále rostoucí škály konektorů (momentálně asi 200) jako jsou SQL Database, Azure Services, Office 365, Salesforce, Google, Twitter a další. Tyto konektory mají různé spouštěče a akce, díky kterým Logic App bezpečně přistupuje k datům a zpracovává je.

## Jak fungují?

Každý Logic App workflow začíná spouští, která je aktivována v případě splnění požadovaných specifik (událost, získání nových dat). Jako spoušť lze využít i časovač, či například HTTP požadavek v předdefinovaném tvaru. Při každém spuštění se vytvoří nová instance aplikace, která provádí jednotlivé akce workflow. Tyto akce mohou pracovat s daty, nebo také kontrolovat průběh – podmínky, smyčky, switche. Logic Apps lze vytvářet skrz vizuální Logic Apps Designer přes Azure portal ve vašem webovém prohlížeči, nebo například z prostředí Visual Studia. Pro větší customizaci je dostupný „code view“ mód, kde naleznete definici Logic App v JavaScript Object Notation (JSON). Také můžete využít Azure PowerShellu, nebo například Azure Resource Manageru. Logic Apps jsou nasazeny a běží kompletně v cloudu na Microsoft Azure.

## Proč je používat?

Díky Logic Apps lze snadno automatizovat pracovní postupy. Jsou vhodné i k provázání různých systémů a Tím pádem se můžete soustředit pouze na funkcionalitu, a nemusíte se vůbec starat o hosting, správu, údržbu nebo monitoring aplikace. Logic Apps obstarají všechno za vás. Navíc platíte pouze za to, co využíváte, jelikož cena se řídí podle consumption pricing modelu (podle spotřeby). V drtivé většině případů není potřeba psát jedinou řádku kódu, pokud však narazíte na komplexnější řešení, kde si nevystačíte s poskytovanými nástroji, tak můžete využít vlastních Azure Functions a pro interakci na jednotlivé eventy v Azure, například Azure Event Grid.

## Praktická ukázka

Pro praktickou ukázku si vytvoříme jednoduchou aplikaci, která bude synchronizovat nové kontakty z Dynamics 365 do MailChimp distribučního listu.

### Vytvoření Logic App

V Azure portálu si vytvoříme novou Logic App. Do vyhledávacího pole napište „logic app“. [![](https://msdnshared.blob.core.windows.net/media/2018/02/138.png)](https://msdnshared.blob.core.windows.net/media/2018/02/138.png) Po vybrání Logic App stačí kliknout na tlačítko Create. [![](https://msdnshared.blob.core.windows.net/media/2018/02/215.png)](https://msdnshared.blob.core.windows.net/media/2018/02/215.png) Vyplňte potřebné údaje a opět klikněte na tlačítko Create. [![](https://msdnshared.blob.core.windows.net/media/2018/02/314.png)](https://msdnshared.blob.core.windows.net/media/2018/02/314.png)

### Logika aplikace

Pro konstrukci logiky aplikace využijeme Designer. Ten najdete po otevření aplikace v levém panelu pod Developer Tools. [![](https://msdnshared.blob.core.windows.net/media/2018/02/414.png)](https://msdnshared.blob.core.windows.net/media/2018/02/414.png) V Designeru máme několik přednastavených šablon, díky jimž se dá vývoj ještě více urychlit. Pro tuto ukázku si ale zvolíme prázdnou šablonu – Blank Logic App Template. [![](https://msdnshared.blob.core.windows.net/media/2018/02/514.png)](https://msdnshared.blob.core.windows.net/media/2018/02/514.png) Nyní už můžeme tvořit konkrétní kroky. Každou Logic App spouští tzv. trigger. V našem případě bude tímto spouštěčem vytvoření nového kontaktu v CRM, tudíž jako konektor zvolíme Dynamics 365. [![](https://msdnshared.blob.core.windows.net/media/2018/02/614.png)](https://msdnshared.blob.core.windows.net/media/2018/02/614.png) Logic App nemá prozatím do vaší instanceDynamics přístup, a proto je nutné poskytnout přihlašovací údaje. Velkou výhodou je, že přihlášení proběhne pouze jednou a aplikace si uloží token, pomocí kterého následně do Dynamics vstupuje. Jak jsem již nastínil, tak naším spouštěčem bude vytvoření kontaktu. [![](https://msdnshared.blob.core.windows.net/media/2018/02/714.png) ](https://msdnshared.blob.core.windows.net/media/2018/02/714.png) V následujícím kroku je potřeba Logic App nastavit, do jaké organizace (v případě že jich je pod jedním účtem více) a k jaké konkrétní entitě má přistupovat. V našem případě se jedná o entitu „Contacts“. [![](https://msdnshared.blob.core.windows.net/media/2018/02/87.png)](https://msdnshared.blob.core.windows.net/media/2018/02/87.png) Pokud očekáváte velký nával požadavků, je lepší interval zkrátit, aby aplikace nemusela zpracovávat tolik záznamů najednou. Naopak pokud nejste tolik vytíženi, můžete interval prodloužit. Přidejte další krok. [![](https://msdnshared.blob.core.windows.net/media/2018/02/95.png)](https://msdnshared.blob.core.windows.net/media/2018/02/95.png) Vyhledejte connector – MailChimp a opět poskytněte aplikaci přihlašovací údaje. Dále zvolte akci „Add member to list“. [![](https://msdnshared.blob.core.windows.net/media/2018/02/106.png)](https://msdnshared.blob.core.windows.net/media/2018/02/106.png) Při kliknutí do pole lze doplnit dynamický obsah (obsah z entity kontaktu z CRM). [![](https://msdnshared.blob.core.windows.net/media/2018/02/1115.png)](https://msdnshared.blob.core.windows.net/media/2018/02/1115.png) Finální aplikace by měla vypadat takto: [![](https://msdnshared.blob.core.windows.net/media/2018/02/1211.png)](https://msdnshared.blob.core.windows.net/media/2018/02/1211.png) Právě jsme si ověřili, že vytvoření Logic App je velmi rychlý a jednoduchý proces. Konkrétně v tomto případě by byl vývoj zdlouhavý a náročný, zatímco díky Azure Logic Apps je to otázkou pár minut. Za zmínku na závěr ještě stojí podobný nástroj Microsoft Flow. Je to jednodušší varianta, který je více zaměřená na běžné uživatele a umožňuje jim automatizaci jejich každodenní práce. Oproti Logic Apps má Flow některé praktické funkce, jako například mobilní aplikaci, spouštěč [Flow Button](https://docs.microsoft.com/cs-cz/flow/introduction-to-button-flows) nebo [schvalovací proces](https://docs.microsoft.com/cs-cz/flow/approve-reject-requests). [![](https://msdnshared.blob.core.windows.net/media/2018/02/139.png)](https://msdnshared.blob.core.windows.net/media/2018/02/139.png)   Z Flow lze dokonce v případě potřeby (pokročilejší možnosti, proces ve správě IT oddělení) vytvořit pomocí exportu šablony Logic App: [![](https://msdnshared.blob.core.windows.net/media/2018/02/144.png)](https://msdnshared.blob.core.windows.net/media/2018/02/144.png)