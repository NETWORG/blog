---
author: Jan Hajek
title: Best practices for managing students in Active Directory
slug: best-practices-for-managing-students-in-active-directory
id: 320
date: '2018-09-03 08:00:19'
categories:
  - English
  - Office 365
  - Windows
tags:
  - Active Directory
  - Azure Active Directory
  - EduLog.in
  - Importing Users
  - Office 365
  - PowerShell
  - SkolniLogin.cz
---

For quite a long time, we have been running a local service called [SkolniLogin.cz](https://skolnilogin.cz) which primarily focused on providing SSO experience for various systems at schools (primary and high schools) along with automatic synchronization with the school's information system. Throughout the time we have hit a lot of edge scenarios, and compiled a best practices guideline.

# Obtaining the data for import

One of the worst parts of the whole job - getting the raw data. I was really surprised when we visited some customers and we learned that they didn't have a single source of truth database where they kept all the student, teacher and employee records. It is probably due to the fact, that none of the current school information systems available on the market here were designed for that. So we usually ended up taking the student data as primary (since it is much easier to create bunch of teachers than hundreds of students).

# Sanitizing the data

So now when we have the data, you would expect them to be homogenous and correct, right? No way! Actually, we have encountered so many issues with the source data that it is close to impossible to have some universal process. For example, some people had trailing spaces in their names, bunch of them (simple trim fixes that), but there were worst. Those people who were filling the data into the system initially made mistakes in spelling, typing etc. This is a very recent one I encountered - _Surname1992_ - basically the HR person at the school typed the year of birth in the surname. Another example would be surname like _Surname (Surname2)_ - what do you make out of that (we got that value in field _Surname_)?! Later I learned that the student got married and that they had their original _Surname2_ in the name so they could easily find them - great, but that isn't the person's legal surname anymore, is it, why do systems implement a _DisplayName_ field?

# Data uniqueness

Next important thing - uniquely identifying the person (you don't have to care about that match if you are maintaining a low number of students, however if you go hundreds and up, it should be rather a requirement). Since the data come from another system, you should usually have a way to match the person in Active Directory to the original data from the system - for easier updates for example. Some systems offer their own unique identifiers - which is great - until you hit a situation, where the unique identifier doesn't uniquely identify a physical person (why would it right?) but probably identifies enrolled person into the class. We have hit this scenario couple of times - which usually result in multiple Active Directory accounts being created for the same user. The solution to this is to use some other identifier - in the Czech Republic, every person is assigned a [national identification number](https://en.wikipedia.org/wiki/National_identification_number#Czech_Republic_and_Slovakia) (something like SSN in US). So then you need the source data to contain that sort of information. You are going to hit both issued mentioned above - you will need to sanitize it and obtain those data. Yet still we have hit issues like there were twins in the system with same national identification number because simply someone made a mistake when entering the data - do those school information systems ever validate any input? And well, other issues like the fact that the school didn't have the identification number of file - yet it is a requirement. Funny, right? Then we need to process the identifier. You are usually going to choose an Active Directory field like _employeeNumber_ or _msDs-cloudExtensionAttribute1_ and so on. There you need to store the value, however, for some reason, the national identification number here is considered a sensitive information. I will try not to go too deep into this here, but basically, if you are doing a business as a self-employed person, the state assigns you an _ICO_ number which is basically your national identification number which you have to fill out basically everywhere and anyone can find it on the government's site - kinda absurd right? Anyways, to store those data, we use following format _IDIssuer,IDType,SHA1(ID)_ which basically partially protects the data but allows us to uniquely identify and match the person while keeping the original number kiiiiinda safe. That way, we can easily match the person in any other required system (usually the original one) by simply executing _SELECT * FROM ... WHERE SHA1(ID) == '<hashedIdFromUsersProfile>'_. Just to give further comment, _IDIssuer_ is usually the country which issued the identified - like _CZ_, _US_ etc. or _INT_ which means the ID was generated by the internal system (as long as it uniquely identifies the person). _IDType_ is another abbreviation _BN_ (birth number or NIN), _SSN_, etc. and then the value itself.

# Generating usernames

Another usually simple part. Each school implements different pattern for their usernames - first.last, last.first, lastfir0, different strategies when duplicates happen .1, .2, .3, lastfir, lastfirs, lastfirst, ... We have a generate which accepts the pattern type and spits out the properly formatted username.

# Deciding on grade IDs

Another thing which differs across schools and countries are classes. This one is probably the most simple one. The majority of schools implement grades like 1\. A, 2\. B, 3\. B, ... system. This is good because when you know the year when the grade begun, you can simply create a unique class ID like this: YYYY-TAG which results in (2018-A, 2017-B, 2016-B). _Just a side note - class in the context of this article = mathematics, biology, history, etc._

# Organizing objects in Active Directory

Before importing users into AD we need to think of some decent structure. Currently we implement following:

*   OU=School
    *   OU=Groups
        *   All Teachers
        *   All Employees
        *   All Students
        *   ...
        *   OU=Classes
            *   2018-X
    *   OU=Students
        *   Student 1
        *   Student 2
    *   OU=Teachers
        *   Teacher 1
    *   OU=Employees
        *   Employee 1

We used to split students into separate organizational units (OU) however when we encountered schools where a student is enrolled into multiple grades, this was hardly possible. OUs are however really great for easy group policy targeting, however with a couple of clicks, you can target a GPO to specific group as well, which makes it simple when you create groups for each grade. With regards to groups, we assign an e-mail address to each group so after it is synced to Office 365 it becomes a mail-enabled security group so teachers can easily communicate information to all grades. All group memberships are direct in our case (but I will get into that later).

# Syncing with Azure AD

Next big step is synchronizing the user's with Azure AD. With Azure AD Connect, things are really simple. All you have to do to sync the users is sign in, choose the proper OUs for syncing (because why sync entire AD?) and you are good to go. Just make sure your UPNs are set right! With UPNs, a lot of schools are choosing to use a 3rd level domain for students - _@student.school.tld_. It kind of adds a visual security - especially when students send outbound e-mails they can be easily distinguished as school's students and are not likely to impersonate the staff.

# Assigning Office 365 licences

This used to be a messy process - you usually had a PowerShell script and had to remember to run it to assign the licenses. Nowadays, however you should switch to [group-based licensing](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-licensing-whatis-azure-portal), which is the reason why we do most group memberships direct. Group-based licensing works only with direct membership at the moment. Thanks to it, you can easily assign licences to hundreds of people without leaving Active Directory.

# Wrap up

So this is the basic process of enrolling new students to school's Active Directory. Once you have the user's synced to Azure AD, you can unlock bunch of other scenarios like [SSO and integration with Moodle](https://docs.moodle.org/34/en/Office365), RADIUS / eduroam and so on. The life of an IT admin at school isn't easy, however we are trying to make it as comfortable as we can.