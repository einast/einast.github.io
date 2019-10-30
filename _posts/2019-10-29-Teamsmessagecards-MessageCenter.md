---
layout: post
title: Teams Message Cards - Part II (Office 365 Health Status)
image: /images/TeamsMessageCenter.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

This is part III of the series Teams message cards.

You'll find the previous parts here:
 [Part I](https://thingsinthe.cloud/Teams-message-cards-Getting-Office-365-roadmap-into-channel/)
 [Part II](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/)

In the third part, we'll use the script from the second part, with some adjustments. The source of the data is the same, the difference is that we look for message type **MessageCenter** instead of **Incident**. The goal is to get the updates from the Microsoft Message Center into a Teams channel. It is useful to know what Microsoft plans that affects your tenant. The script will be run on a schedule (default is once per day, but can be run as often as required).

The script will produce a nice looking message cards, and if any links are embedded in the update, it will be added as buttons.

![](/images/TeamsMessageCenter.PNG)

As this data is not available without logging in to the Office 365 portal, we need to either create an authentication mechanism that handles that for us, or reuse the credentials created in [Part II](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/) as we will get data from the same source.

**High-level steps:**
1. Authentication: 
    - Either create an Azure Enterprise App, give the necessary access to read health status (see previous post on how to set it up)
    - Reuse the credentials created in the second post.
2. Set up an incoming webhook
3. The script: Set the variables in the script according to your environment
4. Test: Run the script to verify
5. Automation: Set the schedule using Azure Automation


**1. Authentication**
To keep it simple, I will reuse the credentials from part II, as we will query the same data source.

**3. The script**
- Download the script from [Github](https://github.com/einast/PS_M365_scripts/blob/master/M365MessageCenterUpdates.v2.ps1)

- Adjust the user variables:
```
$ApplicationID = 'App ID'
$ApplicationKey = 'App key'
$TenantDomain = 'tenant domain' # Alternatively use DirectoryID if tenant domain fails
$URI = 'Teams webhook URI'
$Now = Get-Date
$Hours = '24'    
$color = '0377fc'
```
The script is almost a blueprint of the previous. 

The difference is that we want message type:
```powershell
$incidents = $messages.Value | Where-Object {$_.MessageType -eq 'MessageCenter'}
```
and **not**:
```powershell
$incidents = $messages.Value | Where-Object {$_.MessageType -eq 'Incident'}
```
Like in the previous script, the variables you need to adjust are these:

**\$ApplicationID** - from the generated application in step 1, or reused from Part II

**\$ApplicationKey** - from the generated application in step 1, or reused from Part II

**\$TenantDomain** - your FQDN primary domain

**\$URI** - the generated URI from the creation of the incoming webhook

**\$Hours** - The last x hours the script will look for updates. Default in the script is 24 hours, as I run the script on that schedule.

**\$color** - The color of the card in HTML code. I set mine to blue, but feel free to adjust, or simply go without any color coding. In this script I choose the same color for all cards.

The script will request new or updated feature announcements by using the provided application ID and key. 



It will then create one or more payloads (if there are any available within the time frame), and post in the Teams channel.

**4. Test**

Do a dry run to see of the script works as intended with your own variables. If you see no incidents posted, it could be because of the time interval. Try to expand the number of hours to see if you get any hits.

Once your happy, revert the interval and you're ready to go.

**5. Automation**

I prefer to use Azure Automation, but feel free to pick any scheduling method you need.

Part **IV** is next...