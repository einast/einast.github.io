---
layout: post
title: Teams message cards - Update to Graph API
image: /images/TeamsMessageCenter.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

A couple of years ago, I created a set of script to pull information from Microsft 365 into Teams as message cards.

Due to my current role and assignment, I have not used these scripts myself in a very long time.

It was brought to my attention that the scripts did not work anymore. I just had to look into that, and quickly saw that the API I used did not work any longer. The reason was that the API had been deprecated. For example [this blog post by Tony Redmond](https://practical365.com/moving-office365-service-communications-api-graph/) describes more about it. It was deprecated a few weeks back on December 17th 2021.

Since some people still use these scripts, I thought it would be great if I could help out by fixing them. So I updated them with the new Graph API, and they now seem to work as they used to.

I have pushed the updated code to [my GitHub repo](https://github.com/einast/PS_M365_scripts) where you can find the updated scripts.