---
layout: post
title: Teams Message Cards - Part I (Microsoft 365 Roadmap)
image: /images/TeamsRoadmapWebHook3.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

Microsoft Teams has become an important part of the everyday worklife. It is a great collaboration tool for companies, providing files, chat and telephony. To extend even further, it provides a lot of possibilities for integration with other services. 
I would like use those possibilities to present information from external sources in Teams channels. First out was the wish to gain insight into Microsoft's M365 Roadmap from Teams.

Basically all new entries from the [Microsoft 365 Roadmap](https://www.microsoft.com/en-us/microsoft-365/roadmap) should go in to a Teams channel for this purpose.

As the same URL provides an RSS feed, I can use Powershell to:
- Get data (I would like to do this once per day)
- Parse the data, select the parts I want
- Adapt the output as JSON in a supported Message Card format
- Post it in the Teams channel by using an incoming webhook

Further, I would like the message cards to be color coded based on the status of the feature:
- Red: In development
- Yellow: Rolling out
- Green: Launched

 This resulted in a short script to provide the required functionality.

![](/images/TeamsRoadmapWebHook3.PNG)

#### Setup ####

- Set up an [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/connectors/connectors-using) in your selected Teams channel
- Copy the URI that the connector generated (you need that later)
- Download the script from [here](https://github.com/einast/PS_M365_scripts/blob/master/M365RoadmapUpdates.ps1)
- Adjust the user defined variables to fit your environment:
    - **$URI**: The URI you generated when you set up the incoming webhook
    - **$Hours**: As default, the script will look for any updates the last 24 hours.
- Do a dry-run to see if anything is posted. If not, try to adjust the **$Hours** variable to a higher value
- When you're happy, schedule the script to run on a schedule that aligns with your **$Hours** variable:
    - Being cloud-first, Azure automation is what I use

Parts **II**, **III** and **IV** will follow shortly.
