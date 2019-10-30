---
layout: post
title: Teams Message Cards - Part IV (Azure Health Status)
image: /images/AzureHealth.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

This is part IV of the series Teams message cards.

You'll find the previous parts here:
 [Part I](https://thingsinthe.cloud/Teams-message-cards-Getting-Office-365-roadmap-into-channel/)
 [Part II](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/)
 [Part III](https://thingsinthe.cloud/Teamsmessagecards-MessageCenter/)

This is the last part of the series. This time we'll get current service health events in the Azure subscription.  The script will be run on a schedule (default is every 12 hours, but can be run as often as required).

The script will produce a nice looking message cards, and a link Azure Health.

![](/images/AzureHealth.PNG)

As this data is not available without logging in to the Azure Resource Health API. We need to either create an authentication mechanism that handles that for us, or reuse the credentials created in [Part II](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/). These credentials need **Reader** role on the subscription in order to get the data.

Reference: [Azure Resource health API](https://docs.microsoft.com/en-us/rest/api/resourcehealth/)

![](/images/AzureHealthAccess.png)

**High-level steps:**
1. Authentication: 
    - Either create an Azure Enterprise App, give the necessary access to read subscription data (see previous post on how to set it up)
    - Reuse the credentials created in the second post.
2. Set up an incoming webhook
3. The script: Set the variables in the script according to your environment
4. Test: Run the script to verify
5. Automation: Set the schedule using Azure Automation


**1. Authentication**
I choose to create new credentials, as the requirement is **Reader** on the subscription.

**3. The script**
- Download the script from [Github](https://github.com/einast/PS_M365_scripts/blob/master/AzureHealthStatus.ps1)

- Adjust the user variables:
```powershell
# User defined variables
$ApplicationID = 'application ID'
$ApplicationKey = 'application key'
$tenantid = 'your Azure tenant ID'
$subscriptionid = 'your subscription ID'
$URI = 'Teams webhook URI'
$Hours = '12'
$Now =  (Get-Date).ToUniversalTime()
```

Like in the previous script, the variables you need to adjust are these:

**\$ApplicationID** - from the generated application in step 1, or reused from Part II

**\$ApplicationKey** - from the generated application in step 1, or reused from Part II

**\$tenantid** - your tenant ID

**\$subscriptionid** - your subscription ID

**\$URI** - the generated URI from the creation of the incoming webhook

**\$Hours** - The last x hours the script will look for updates. Default in the script is 12 hours, as I run the script on that schedule.

The script will request new or updated events.

It will then create one or more payloads (if there are any available within the time frame), and post in the Teams channel.

**4. Test**

Do a dry run to see of the script works as intended with your own variables. If you see no incidents posted, it could be because of the time interval. Try to expand the number of hours to see if you get any hits.

Once your happy, revert the interval and you're ready to go.

**5. Automation**

I prefer to use Azure Automation, but feel free to pick any scheduling method you need.

This was the last post in the series. I hope you found it useful. Any feedback is welcome, please feel free to reach out.