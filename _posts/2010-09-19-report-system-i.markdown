---
layout: post
title:  "Report System Part I"
date:   2010-09-19 18:00:00
categories: programming technology-issues
---

About a year and a half ago one of our customers told us about their new requirement. They wanted some kind of a report system for handling their letters and reports, they have some letters and reports with fixed contents(for specific kinds) and they want some parts of those would be interchangeable like Name, Letter No, Title and etc so they can just insert their special values for each letter or report then print it in their desirable format.

so after some searches we decided to use **Crystal Report** (some kind of a report handling technology that's suitable for .NET apps). By the way I hate Crystal Report (it's so stupid, specially for languages with **right to left writing style**), I spent a lot of time on these letters but they weren't what we expected. so we manipulate them very much and after quite two weeks they were almost fine(but there were still some problems like position of lines in the page and etc).

everything was almost good till we **switched** our systems to **VS2010**, at this time Crystal Report (**old version**) and VS2010 had some problems with each other and report system was ruined. so we got last version of Crystal Report and try it in the application but there was still some version compatibility issues(also there were other stuff which it's not necessary to talk about them here cause you got the point, fair enough). we struggled with these problems so much but we couldn't solve them clearly and perfectly.

after a while,  one day my boss came to me and said : "**why don't we write our own report system?**", that sounded like a very interesting and exciting idea so I nodded my head while I was thinking about how cool is that? so I agree with that(well he's my boss and I couldn't disagree even if I wanted to but I was honestly agree with him though:D). after a lot of discussion over this idea we decided to write an html page for each letter or report for it's fixed content and having some special pattern for interchangeable parts, so our application can read through that page and find them for replacing actual values(which are determined by user) with them. with this way our users can edit the format and style of these pages with any kind of html-editor they want too.

we started to think about the **design** of this system and **modules** that we need for it and what **tests** we should have and other stuff like these which I'm gonna talk about them in more **details** in the *next part* ...
