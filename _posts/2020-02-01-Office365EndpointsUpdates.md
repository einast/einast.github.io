---
layout: post
title: Office 365 Endpoint Updates - Teams message cards
image: /images/O365Endpoints01.PNG
categories: [Powershell, Office 365, Microsoft Teams, Azure Automation]
---

Approximately once every month, Microsoft updates their Office 365 endpoints. These are the internet endpoints for all the Office 365 services you need to be aware of if you filter/block outbound traffic.

The complete list is [here](https://docs.microsoft.com/en-us/office365/enterprise/urls-and-ip-address-ranges) (worldwide endpoints). If you use one of the other Office 365 clouds (21Vianet, Germany, US DOD or US government), links to those lists are provided in the above link. Also in those specific pages, you will find change log RSS feeds that this script uses. 

These endpoints are categorized based on their importance, more info [here](https://aka.ms/pnc).

Working for customers with high security requirements using whitelisting, this might be handled automatically. Many proxy server and firewall vendors have features to ensure that rules are updated once Microsoft publish these updates. But not all. In addition, these vendor features are not always functioning 100% in my experience. Keeping rules updated ensures that traffic flow as expected so customers avoid any related issues.

As I have been finding use cases for getting useful information into Teams channels the last months, I created a script (two actually, one generic and one for Azure Automation) to help keeping up to date. These scripts will check Microsoft's RSS daily (or as often as you like), and if any updates, post to a Teams channel.

As usual replace the variables with your own. The ones you need to change are **\$URI**, and **\$Hours** if you want to check for updates other than every 24 hours. **\$EndpointsUpdates** is the RSS feed for *Worldwide* endpoint updates (please remember to change if you use one of the other Office 365 clouds). **\$EndpointURL** is Microsoft's O365 endpoint documentation.

```powershell
$URI = 'https://your.Teams.channel.webhook.URI'
$EndpointsUpdates = 'https://endpoints.office.com/version/worldwide?allversions=true&format=rss&clientrequestid=b10c5ed1-bad1-445f-b386-b919946339a7'
$Hours = '24'
$Now = Get-Date
$EndpointURL = 'https://docs.microsoft.com/en-us/office365/enterprise/urls-and-ip-address-ranges'
```

**I also have a version for Azure Automation, the variables are similar, but you'll need to create them as Azure Automation variables.**

I formatted the message cards in the following way:

- A short change description
- Datestamp of the change
- A link to the change (although in JSON, it will tell you what has changed, like the following example):

```json
[
  {
    "id": 500,
    "endpointSetId": 149,
    "disposition": "Add",
    "version": "2020012800",
    "impact": "AddedUrl",
    "current": {
      "expressRoute": "false",
      "serviceArea": "Common",
      "category": "Default",
      "required": "true",
      "tcpPorts": "80,443"
    },
    "add": {
      "effectiveDate": "20191125",
      "urls": [
        "workplaceanalytics.cdn.office.net",
        "workplaceanalytics.office.com"
      ]
    }
  }
]
```

Here the card tells that there have been some changes, like two added URLs, requiring communication on ports 80 and 443, category Default (previous category, required).

- A link to the Microsoft endpoint list mentioned at the beginning of this post

The final result will look like this when posted to Teams:

![](/images/O365Endpoints01.PNG)

I might refine the output later, but for now it fulfills the goal of getting notified of changes.

The scripts are located here:
- [Azure Automation version](https://github.com/einast/PS_M365_scripts/blob/master/AzureAutomation/O365EndpointUpdatesAzureAutomation.ps1)
- [Generic version](https://github.com/einast/PS_M365_scripts/blob/master/O365EndpointUpdatesGeneric.ps1)