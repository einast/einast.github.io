---
layout: post
title: Teams Message Cards - Posting useful information
image: /images/TeamsO365Health.png
categories: [Powershell, Office 365, Microsoft Teams, Azure Automation]
---

Microsoft Teams has become an important part of the everyday worklife. It is a great collaboration tool for companies, providing files, chat and telephony. To extend even further, it provides a lot of possibilities for integration with other services. Incoming webhooks is a way to integrate with other services, for example to present information.

In this series of posts, I will explore Powershell and webhooks to post useful information into Teams channels.

![](/images/TeamsMessageCenter.PNG)

Not being a Powershell guru, my goal was to have a template script that I easily could adapt to different use cases. As new ideas popped up, I choose to split up in separate posts.

- [Part I - Microsoft 365 Roadmap](https://thingsinthe.cloud/Teams-message-cards-Getting-Office-365-roadmap-into-channel/): This is an introduction on how to get info from the Microsoft 365 Roadmap RSS feed. It is the first iteration of the script that was further developed to other use cases.
- [Part II - Office 365 Health Status](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/): Further development of the script to present incident information from your Office 365 tenant
- [Part III - Microsoft Message Center - Announcements](https://thingsinthe.cloud/Teamsmessagecards-MessageCenter/): This script is similar to the previous, but with some adaptions to present announcements for your Office 365 tenant.
- [Part IV - Azure Resource Health](https://thingsinthe.cloud/AzureHealth/): This last post is about presenting service outage/incidents from your Azure tenant.

I added links to the other two scripts created after the first series for reference here as well:
- [Additional script #1 - Office Pro Plus updates announcements](https://thingsinthe.cloud/OfficeProPlus-updates/): Presenting Office Pro Plus change log information
- [Additional script #2 - Office 365 Endpoint updates](https://thingsinthe.cloud/AzureHealth/): Keeping updated on Office 365 endpoint changes