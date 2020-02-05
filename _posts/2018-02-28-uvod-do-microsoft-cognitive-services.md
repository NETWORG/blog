---
author: Matej Samler
title: Úvod do Microsoft Cognitive Services
slug: uvod-do-microsoft-cognitive-services
id: 100
date: '2018-02-28 06:55:49'
categories:
  - Azure
tags:
  - AI
  - Cognitive Services
---

Microsoft Azure nabízí spoustu praktických pomůcek, které můžete využít ve vašich aplikací. Mě osobně nejvíce zaujaly „poznávací“ služby, tedy Cognitive Services. Tyto služby jsou postaveny na strojovém učení a umělé inteligenci. Zjednodušeně lze postup popsat tak, že se počítač může na základě příkladů natrénovat a naučit rozpoznávat přesně to, co potřebujete, aby vyhodnotil. Není tedy problém, aby na fotografii našel obličeje, porozuměl vašim slovům, a nebo z textu poznal, jestli jste byli spokojení s obsluhou v restauraci. Vy poté můžete tyto funkce využít ve vaší aplikaci bez nutnosti tak složitou funkcionalitu implementovat a neurální síť trénovat, například k odemykání pomocí obličeje nebo vytvoření chat bota, který bude vašim zákazníkům odpovídat na otázky a v případě potřeby se obrátí na lidského operátora. Já jsem se podíval na práci s textem a řečí a na jejich jednotlivá rozhraní, která jsou přístupná vývojářům.

# Získání přístupu

Získání přístupu k těmto funkcím je velmi jednoduché. Stačí se zaregistrovat pomocí vašeho Microsoft účtu (lze také použít například Facebook nebo Twitter), a poté si vybrat kterou službu chcete používat. Každá služba nabízí hned několik možností předplatného a vy si můžete vybrat podle velikosti vaší aplikace. ![](/uploads/2018/04/cognitive1.png) Já jsem si pro začátek vybral verzi zdarma, která se skvěle hodí na testování a nabízí až 5 000 transakcí měsíčně. Cognitivní služby nejsou jeden velký balíček, ale je to několik funkcí, které jsou nabízeny samostatně. Zaregistroval jsem si například klíče k používání API pro text a pro rozpoznávání řeči (tedy Speech-to-text) ![](/uploads/2018/04/cognitive2.png) [](https://msdnshared.blob.core.windows.net/media/2018/02/216.png) Po registraci dostanete dva klíče, oba ovšem plní tutéž funkci.

# Analýza textu

Začneme tedy s analýzou textu. Tato API nabízí 3 hlavní analytické funkce. První z nich je rozpoznání jazyka, ve kterém je text napsán, druhý je extrakce nejdůležitějších slov či frází z textu a nakonec číslo (mezi nulou a jedničkou), které vyjadřuje jaký sentiment tento text vyjadřuje, tedy pokud je zpráva pozitivní nebo negativní. Microsoft sám nabízí velice přehledný příklad, na kterém je velice jednoduché se naučit toto API používat. Nejprve je samozřejmě důležité umět s API komunikovat, což je velmi jednoduché, jelikož vám stačí stáhnout příslušný balíček z Nugetu. ![](/uploads/2018/04/cognitive3.png) Je velice jednoduché vytvořit si klienta na rozpoznávání textu. Jediné, co je potřeba, je určit jeho region (který si zvolíte při registraci klíče) a klíč samotný. ![](/uploads/2018/04/cognitive4.png) ![](/uploads/2018/04/cognitive5.png) Tato jednoduchá funkce zjistí, v jakém jazyce je váš text napsaný. Výsledek je ve formátu ISO6391, tedy například "en", "jp" nebo "cz". Analýza textu dokáže rozpoznat až 120 různých jazyků z celého světa. Ostatní dvě funkce jsou bohužel omezeny jen na několik málo jazyků. Například o české větě by vám řekla, že je česky, ale důležité fráze z ní bohužel nevybere. Jak jste si mohli všimnout, tak věty pro tuto funkci se posílají jako seznam. Můžete tedy poslat několik vět s různými id najednou a analýzu všech dostat zpátky ve formátu JSON okamžitě, místo analýzy každé věty zvlášť. Ve formátu JSON lze také data odesílat. ![](/uploads/2018/04/cognitive6.png)  

# Analýza textu

S tímto API se mi pracovalo velice dobře. Bylo skvěle vysvětleno, je jednoduché na používání a výsledky byly přesně to, co jsem hledal. Myslím, že má mnoho využití. Rozpoznání jazyka se dá používat například v překladačích (což je mimochodem další funkce cognitivních služeb), rozpoznání sentimentu je výborné, pokud chcete například automaticky vědět, jestli recenze na restauraci byla pozitivní nebo negativní, aniž by to musel host sám určovat, a nakonec výběr klíčových frází je ideální pro chat boty, kteří díky tomu rychle rozpoznají, co po nich zákazník může chtít. S touto službou se mi pracovalo tak dobře, že jsem na tom chtěl dále stavět a propojit ji s jinou službou.

# Rozpoznávání řeči

Cognitive Services API, které pracuje s řečí, nabízí hned několik různých funkcí, například překládání řeči do jiného jazyka nebo rozlišování a rozpoznání hlasů různých lidí. Já jsem se soustředil nejvíce na funkci, která dokáže převést mluvená slova na text, a zpátky text na mluvenou řeč. Původní plán mé aplikace byl, že uživatel namluví krátkou větu do mikrofonu, cloud service větu zpracuje na text, a poté použije textovou analýzu, o které jsem mluvil výše, a zobrazí se výsledek. Bohužel tady jsem narazil na jistý problém.

## Dokumentace

Stejně jako u práce s textem je i zde ke stažení aplikace jako příklad. Bohužel je tato aplikace to jediné co Microsoft nabízí. Nikde jinde jsem na jejich stránkách nenašel popis metod nebo konstant, které jsou v součástí tohoto API. Popis funkčnosti je tedy spíše teoretický než praktický. ![](/uploads/2018/04/cognitive7.png) Praktická ukázka aplikace funguje skvěle. Ukazuje všechny funkce rozpoznávání řeči ve své kráse, včetně průběžných výsledků a možnosti poslouchat buďto z mikrofonu nebo ze zvukového souboru. Bohužel je tato aplikace docela komplexní, takže není tak jednoduché z ní rychle a jasně pochopit, co je třeba udělat, abyste tuto funkčnost zprovoznili i sami na své aplikaci, a jelikož je to jediný zdroj zdrojového kódu, který tyto metody používá, není tak jednoduché si práci osvojit jako v minulých případech. ![](/uploads/2018/04/cognitive8.png)

# Závěr

Cognitive Services jsou velmi praktickou pomůckou pro vytváření moderních IT scénářů. Nabízí další možnosti, jak uživatelům zpříjemnit a zjednodušit používání vaší aplikace. Je jen škoda, že pro lidi, kteří nemají ještě s používáním této technologie tolik zkušeností, není dostatek materiálu na zpřístupnění i těch komplexnějších funkcí.