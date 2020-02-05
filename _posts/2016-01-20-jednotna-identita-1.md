---
author: tomas.prokop
title: 'Jak na jednotnou identitu: 1. část - Adresářové služby'
slug: jednotna-identita-1
id: 158
date: '2016-01-20 20:48:33'
categories:
  - IT Pro
tags:
  - Active Directory
  - Azure Active Directory
  - Google Apps
  - SSO
---

Cílem této série je je přiblížení situace a dostupných prostředků k sjednocení elektronické identity uživatelů softwaru, který spravuje školní instituce. Sjednocením je možné optimalizovat správu uživatelských entit a eliminovat redundantní vyplňování uživatelských dat, které je rizikové kvůli chybám zaviněným lidským faktorem. Takového stavu je možné docílit pomocí konsolidace dat do jednoho zdroje, využitím dostupných ověřovacích a adresářových protokolů a automatizací úkonů potřebných k jejich správě.   ![ji_blog](/uploads/2016/01/ji_blog.png)    <span style="text-decoration: underline">Jak na jednotnou identitu:</span>

*   [1\. část - Adresářové služby](https://blog.skolnilogin.cz/jednotna-identita-1/)
*   [2\. část - Protokoly](https://blog.skolnilogin.cz/jednotna-identita-2/)
*   3\. část - Nástroje pro správu

  Vycházím ze zkušeností získaných po úspěšné prvotní fázi projektu SkolniLogin.cz, která zahrnovala průzkum trhu, vývoj externí služby, implementaci na pilotních školách, proškolení učitelů a správců, zpracování zpětné vazby a dat z prvních měsíců produkčního provozu. Její části mohou být užitečné především pro správce, kteří budou řešit propojení systémů zde zmíněných. Výčet dostupných technologií není kompletní a jsou zde zmíněny pouze vybrané a nejrozšířenější. Na začátek je nutné definovat dva možné způsoby, jak docílit sjednoceného přihlášení více  nezávislých aplikací.

*   **Single Sign-On** – Aplikace přesměruje uživatele na federační server a ten se postará o autentizaci. Následně pošle zpět informaci o výsledku a údaje přihlášeného uživatele. Nevýhodou je, že federační server se stává kritickým prvkem a je nutné zajistit jeho dostupnost. Při výpadku nebo útoku není možné používat závislé systémy.
*   **Same Sign-On** – Údaje, tj. uživatelské jméno a heslo, pro přihlášení jsou identické a jsou pravidelně synchronizovány. Pravidlem je, že uživatel musí zadávat stejné údaje na různých místech. Je to kompromis, který je mnohem jednodušší implementovat, ale uživatelská zkušenost není úplně přímočará. Pro obě možnosti je běžně využívané stejné pojmenování Single sign-on (zkratka SSO). To znamená, že je velmi pravděpodobné, když se někde vyskytne zmínka o druhé metodě, tak bude chybně nazvána též Single Sign-On.

Velmi omezené množství aplikací využívá tzv. Single Sign-Out proces, který je ale neméně důležitý a poskytuje způsob, jak se naráz ze všech míst odhlásit.  

## <span style="text-decoration: underline">Zdroje uživatelů / Adresářové služby</span>

Na následujících řádcích naleznete rychlý přehled tří takových služeb, které už pravděpodobně znáte.  

### Proprietární řešení

Často vývojáři softwaru implementují vlastní řešení pro jejich a jejich účty pomocí relačních databází. V některých případech je využíván zmíněný proces Single Sign On k propojení externích účtů s účty v databázi. Některé větší internetové nebo intranetové aplikace využívají adresářové služby, které nejčastěji slouží právě k centralizaci uživatelských dat.  

### Active Directory

Distribuovaná výkonná adresářová služba od Microsoftu, která je součástí systému Windows Server. Nejčastěji se využívá pro správu počítačové sítě s operačními systémy Windows. Umožňuje správcům do značné míry přizpůsobit chování klientů pomocí skupinových politik. Slouží jako katalog informací o objektech v síti (uživatelé, počítače, tiskárny). Objekty jsou umístěny v kontejnerech, které mohou být reprezentovány organizačními jednotkami. <span style="text-decoration: underline">Role a technologie:</span>

*   **AD Certificate Services** – vytváření, distribuce a správa certifikátů veřejného klíče
*   **AD Domain Services** – komunikace mezi uživateli a doménou
*   **AD Federation Services** – webové SSO
*   **AD Lightweight Directory Services** – implementace LDAP
*   **AD Rights Management Services** – ochrana informací a dokumentů před neoprávněným přístupem

Více o možnostech správy se dozvíte v dalším dílu.  

### Azure[**[1]**](#_ftn1) Active Directory

Vysoce dostupná cloudová[[2]](#_ftn2) služba, která se velmi rychle rozvíjí. Je možné využívat variantu, která je zdarma. Z vnějšího pohledu se může jevit jako Active Directory, kterou hostuje Microsoft místo zákazníka, ale pravdou je, že se tyto dvě technologie velmi liší. AAD například nepodporuje klasické doménové politiky, způsobem známým v AD. K přesunu AD do Azure cloudu slouží řešení Azure AD Domain Services. AAD velmi zaměřená na vývojáře aplikací. Poskytuje jim rychlou a prověřenou platformu, kterou je snadné implementovat i do již existujícího softwaru. Správci mohou udělovat přístupy uživatelům do tisíců dostupných a kompatibilních aplikací (např. Office 365, Google Apps). Častým scénářem je propojení s lokální Active Directory. Integraci lze provést pomocí dvou způsobů.

1.  **Synchronizace adresáře a hesel (Same Sign-On)** – Jednodušší způsob, který synchronizuje účty z AD do AAD. Je nutné na server nainstalovat nástroj Azure AD Connect, který zajistí propagaci změn účtů a zabezpečeným způsobem zapíše hash NTLM hashe hesel do AAD, aby bylo možné se přihlásit stejným heslem, ale byl eliminován únik skutečného hesla.
2.  **Federace (Single Sign-On)** – V tomto případě je řešení mnohem náročnější a kriticky závislé na serverech instituce. V rámci AD je nutné zprovoznit AD Federation Services server, Web Application Proxy a Azure AD Connect. Ve výsledku bude ADFS server publikován do internetu a AAD bude uživatele při přihlašování přesměrovávat na něj. Pokud se k webové aplikaci přistupuje přes počítač ve stejné AD doméně, proběhne přihlášení automaticky účtem aktuálního uživatele počítače.

Metody sjednocení přihlašování:

*   **Federované SSO** – Aplikace využije příslušné protokoly a přesměruje uživatele na přihlašovací stránku AAD nebo dále na ADFS.
*   **Password-based SSO** – AAD uchovává heslo do aplikace a pomocí doplňku pro prohlížeč vykoná přihlášení místo uživatele. Tento způsob nevyžaduje úpravy v aplikaci. Hesla může spravovat administrátor nebo uživatelé sami. To může být užitečné třeba, když je potřeba zaměstnancům předat přístup k účtům sociálních sítí bez toho, aby zjistili heslo. Existuje způsob, jak heslo zjistit, ale pro běžného uživatele to není nic jednoduchého.
*   **Existující SSO** – Pokud aplikace podporuje nějaký jiný způsob jednotného přihlášení, je možné aplikaci přidat a zobrazit na ni odkaz z Office 365 a portálu access panel.

U aplikací, které podporují user provisioning nebo protokol SCIM[[3]](#_ftn3), je možné automaticky propagovat změny uživatelů v AD (např. Google Apps). Některé další dostupné funkce:

*   **Multi-Factor Autentizace** – Zvýšení bezpečnosti přihlašování pomocí kombinace „něco, co uživatel ví + něco, co uživatel vlastní.“ V praxi to například znamená, že po zadání hesla přijde na mobilní telefon uživatele notifikace, která oznámí, že probíhá přihlášení a je nutné jej potvrdit. Tato možnost je dostupná pro uživatele Office 365 a placenou verzi Azure Active Directory.
*   **Registrace zařízení** – Po přidání zařízení (Windows, Android nebo iOS) do služby, se do něj zavede uživatelský účet. To může být využito k definici podmíněného přístupu k prostředkům. V kombinaci s řešením správy mobilních zařízení (např. Microsoft Intune) je možné na zařízení vynutit politiky, které mohou zabránit úniku dat apod.
*   **Self-service správa skupin a hesel** – Uživatelé si mohou sami resetovat heslo, pokud jej zapomenou a mají správně nastavené kontaktní údaje. Samostatně lze vytvářet i skupiny uživatelů, které mohou být užitečné třeba v Office 365.

Uživateli velmi vítanou novinkou je Microsoft Passport. Ve Windows 10 nahrazuje zadávání hesla dvoufaktorofou autentizací, která zahrnuje zařízení připojené do AAD (je kompatibilní i s AD, Microsoft účtem a službami podporujícími FIDO[[4]](#_ftn4)) a biometrický prvek (snímání duhovky, otisk prstu, tvar obličeje) nebo PIN. Microsoft Passport též umožňuje přihlášení k počítači provést pomocí mobilního telefonu bezdrátově pomocí technologie Bluetooth. Ve spojení s API[[5]](#_ftn5) Web Account Manager ve Windows 10, které slouží k automatickému přihlášení k webové aplikaci pomocí účtu uloženého v počítači, je možné se přihlásit k počítači třeba otiskem prstu a následně být automaticky přihlášen i do webových aplikací, které v tomto případě využívají AAD.  

### Google Apps Directory

Uživatelé pro Google Apps. Účty se dají spravovat v administrátorské konzoli a skrze API. Lze také nastavit multi-factor autentizaci. Díky funkci OneClick SSO v AAD je možné pomocí protokolu SAML a integrace s Google Apps Directory API delegovat přihlašovací proces směrem k AAD.   Příště budeme pokračovat představením dostupných protokolů a jejich možnostmi. Máte nějaké dotazy ohledně této problematiky nebo máte tip na článek s tématem, o kterém se chcete dozvědět více? [contact-form to='tomas.prokop@thenetw.org' subject='Blog form'][contact-field label='Email' type='email' required='1'/][contact-field label='Co vás zajímá?' type='textarea' required='1'/][/contact-form]

* * *

  [[1]](#_ftnref1) Microsoft Azure = platforma v datacentrech Microsoft [[2]](#_ftnref2) Cloud = provoz serverů v datacentru někde v internetu [[3]](#_ftnref3) [http://www.simplecloud.info/](http://www.simplecloud.info/) [[4]](#_ftnref4) Fast ID Online - [https://fidoalliance.org/](https://fidoalliance.org/) [[5]](#_ftnref5) API = rozhraní pro programování aplikací