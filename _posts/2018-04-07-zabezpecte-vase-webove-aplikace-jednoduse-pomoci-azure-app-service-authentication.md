---
author: Jan Skala
title: >-
  Zabezpečte vaše webové aplikace jednoduše pomocí Azure App Service
  Authentication
slug: >-
  zabezpecte-vase-webove-aplikace-jednoduse-pomoci-azure-app-service-authentication
id: 76
date: '2018-04-07 15:00:34'
categories:
  - Azure
tags:
  - Azure Active Directory
  - Azure App Service
  - MediaWiki
  - SSO
---

Zabezpečte jednoduše vaší aplikaci bez nutnosti většího zásahu do backendu. App Service Authentication vám pomůže zabezpečit váš web nebo mobilní aplikaci. Toho je dosaženo prostřednictvím federované identity. Uživatel se tedy nepřihlašuje oproti vaší aplikaci, ale je přesměrován na třetí stranu (poskytovatele identit), která jej přihlásí a následně je uživatel přesměrován zpět. Vaše aplikace tedy nemusí uchovávat údaje o identitě uživatele. Pro přihlášení do vaší aplikace si můžete vybrat více poskytovatelů identit.

## Poskytovatelé identit, které je možné využít

![](/uploads/2018/04/image001.png)

Kromě výše zmíněných můžete integrovat jiného poskytovatele nebo svoje [vlastní řešení](https://docs.microsoft.com/en-us/azure/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk#custom-auth).

Uživatelé webové aplikace obdrží cookies a zůstanou přihlášeni během používání aplikace. V případě mobilní nebo jiné ne-webové platformy obdržíte JWT (JSON Web Token) token. Pro mobilní řešení existují knihovny, které spoustu práce řeší za vás.

# Jak to funguje

Abyste mohli tuto funkcionalitu používat, musíte nejprve zajistit, aby poskytovatel identity o vaší aplikaci věděl. Poskytovatel vám pak přiřadí id aplikace a klíč, které použijete. Veškerá autentizace se provádí na straně poskytovatele identit. Toto řešení je ideální pro mobilní aplikace, kde kromě autentizace žádný backend nepotřebujete nebo pro malé, převážně statické weby.

# Praktická ukázka

Pro praktickou ukázku si založíme instanci MediaWiki (kterou můžeme například využít jako interní znalostní bázi ve firmě), a zabezpečíme jí pomocí Azure Active Directory. To vše bez znalosti kódu nebo modifikace samotného jádra MediaWiki.

## Vytvoření MediaWiki

V Azure portálu vtvoříme prázdný Web App a do ní rozbalíme aplikaci MediaWiki.

[![](https://msdnshared.blob.core.windows.net/media/2018/01/image003.png)](https://msdnshared.blob.core.windows.net/media/2018/01/image003.png)

Její archiv stáhneme přímo z [https://www.mediawiki.org/wiki/Download](https://www.mediawiki.org/wiki/Download) pomocí Debug Console v  pokročilých nástrojích App Service (Kudu). Konzole se nachází na adrese https://{appname}.scm.azurewebsites.net/DebugConsole kde {appname} nahradíme názvem naší Web App.

Do konzole zadáme následující příkazy

curl https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz > wiki.tar.gz

tar -xzvf wiki.tar.gz

mv mediawiki-1.30.0 wiki

## Nastavení MediaWiki

1.  Do prohlížeče zadejte adresu https://{appname}.azurewebsites.net/wiki/mw-config/index.php
2.  Proklikejte se přes první dvě stránky
3.  Jako typ databáze pro jednoduchost vyberte SQLlite a zvolte „Pokračovat“ [![](https://msdnshared.blob.core.windows.net/media/2018/01/image005.png)](https://msdnshared.blob.core.windows.net/media/2018/01/image005.png) _Poznámka:_ V produkční verzi webu je mnohem lepší využívat plnohodnotný databázový server, ať už se jedná o MySQL (např. v podání [Azure Database for MySQL](https://azure.microsoft.com/en-us/services/mysql/)), PostgreSQL ([Azure Database for PostgreSQL](https://azure.microsoft.com/en-us/services/postgresql/)) nebo klasický Microsoft SQL ([Azure SQL](https://azure.microsoft.com/en-us/services/sql-database/)).
4.  Nastavte si název wiki, administrátorský účet a dejte „Pokračovat“ [![](https://msdnshared.blob.core.windows.net/media/2018/01/image007.jpg)](https://msdnshared.blob.core.windows.net/media/2018/01/image007.jpg)
5.  Profil uživatelských práv nastavte na: Soukromá wiki [![](https://msdnshared.blob.core.windows.net/media/2018/01/image008.png)](https://msdnshared.blob.core.windows.net/media/2018/01/image008.png)
6.  Nyní už stačí vše doklikat do konce.
7.  Na konci se vám stáhne soubor LocalSettings.php. Nahrajeme jej na server do složky s rozbalenou aplikací MediaWiki. Stačí ho přesunout myší. [![](https://msdnshared.blob.core.windows.net/media/2018/01/image012.jpg)](https://msdnshared.blob.core.windows.net/media/2018/01/image012.jpg)

## Nastavení autentizace

Nyní si můžete vyzkoušet, zda vaše wiki funguje. Stačí přejít na adresu https://{appname}.azurewebsites.net/wiki. Zbývá nám zapnout autentizaci.

## Nastavení rozšíření

1.  Nejprve nainstalujeme rozšíření, podobně jako jsme instalovali samotnou MediaWiki
2.  V DebugConsole v Kudu navigujeme do adresáře D:\home\site\wwwroot\mediawiki-1.30.0\extensions
3.  Zde spustíme příkazy curl https://extdist.wmflabs.org/dist/extensions/Auth_remoteuser-REL1_30-217b5f3.tar.gz > auth.tar.gz tar -xzvf auth.tar.gz
4.  V DebugConsole otevřeme soubor LocalSettings.php[![](https://msdnshared.blob.core.windows.net/media/2018/01/image014.jpg)](https://msdnshared.blob.core.windows.net/media/2018/01/image014.jpg)
5.  Do něj úplně nakonec přidáme wfLoadExtension(‘Auth_remoteuser’);[![](https://msdnshared.blob.core.windows.net/media/2018/01/image015.png)](https://msdnshared.blob.core.windows.net/media/2018/01/image015.png)
6.  Pokud byste, chtěli rozšíření více modifikovat, můžete se inspirovat na [https://www.mediawiki.org/wiki/Extension:Auth_remoteuser](https://www.mediawiki.org/wiki/Extension:Auth_remoteuser).
7.  Nezapomeňte změny v souboru uložit.

## Hotovo

Nyní je vše připraveno. Do aplikace se nedostane nikdo, kdo se úspěšně nepřihlásí oproti naší Azure Active Directory. Všimněte si, že celý proces zprovoznění autentizace nevyžadoval zásah do backendu aplikace (kromě instalace pluginu). Díky App Service Authentication/Authorization můžete také například ověřovat uživatele z několika různých organizací (pro MediaWiki se můžete podívat na [tento plugin](https://github.com/TheNetworg/mediawiki-easyauth)).