---
layout: post
title: Teams Message Cards - Part II (Office 365 Health Status)
image: /posts/TeamsRoadmapWebHook3.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

This is part II of the series Teams message cards.

[Part I](https://thingsinthe.cloud/Teams-message-cards-Getting-Office-365-roadmap-into-channel/) started off easy, by parsing an RSS feed and present the data in a Teams channel.

In this second part, we will dig a little deeper. We will look into how to get service health alerts from Office365 into a Teams channel. The use case is that we would like to be notified if any new or updated alerts are available in Teams, without the need of logging into the Office365 Admin portal. Key data from these alerts will be formatted as message cards, with color coding, and adding buttons if any links are available. The script will be run on a schedule (default is 15 minutes, but can be run as often as required).

As this data is not available without logging in to the Office 365 portal, we need to create an authentication mechanism that handles that for us.

**High-level steps:**
1. Authentication: Create an Azure Enterprise App, give the necessary access to read health status
2. Set up an incoming webhook
3. The script: Set the variables in the script according to your environment
4. Test: Run the script to verify
5. Automation: Set the schedule using Azure Automation


**1. Authentication**
- Log in to the [Azure portal](https://portal.azure.com).
- Find **Azure Active Directory** in the left pane, or search.
- Go to **App registrations**
- Click **New registration**
- Give the application a sensible name, select **Accounts in this organizational directory only**, and click **Register**

![](/images/appreg01.PNG)

- Copy the **Application ID**, we need that for later. Also keep a tab on the **Directory ID** as well, it *might* be required later.
![](/images/appreg02.PNG)

- Click **View API permissions**
![](/images/appreg03.PNG)

-Click **Add a permission**
![](/images/appreg04.PNG)

-In the **Request API permissions** click **Office 365 Management APIs**
![](/images/appreg05.PNG)

- In the **Office 365 Management APIs**, click **Application permissions**
![](/images/appreg06.PNG)

- Expand **ServiceHealth** and select **ServiceHealth.Read**. Click **Add permissions**
![](/images/appreg07.PNG)

- Now we need to remove the permission we don't need. Click the **Microsoft Graph (1), User.Read** and click **Remove permission** in the window that pop up. Confirm by clicking **Yes, remove**
![](/images/appreg08.PNG)

- Now all permissions are in place. Click **Grant admin consent for *yourtenantname*** and confirm by clicking **Yes**

- Now we need to generate a client secret for the script. Click **Certificates & secrets**, and **New client secret**
![](/images/appreg09.PNG)

- Give the secret a sensible name, and set the expiration date (1 or 2 years, or never). Click **Add**
![](/images/appreg10.PNG)

- Copy the secret to a secure place, as this is the last time you are able to. If you forget, you need to generate a new secret.
![](/images/appreg11.PNG)

Now the Application ID and Application Key can be used in the script.

**2. Incoming webhook**
Set up an incoming webhook in the Teams channel of your choice. I created a separate team with a channel for each type of service I would like to present.

Check out [this article](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/connectors/connectors-using) for setting up an incoming webhook in your selected Teams channel.

Copy the URI from the webhook you set up.

**3. The script**
- Download the script from [Github](https://github.com/einast/PS_M365_scripts/blob/master/M365HealthStatus.ps1)

- Adjust the user variables:
```
$ApplicationID = 'application ID'
$ApplicationKey = 'application key'
$TenantDomain = 'your FQDN' # Alternatively use DirectoryID if tenant domain fails
$URI = 'Teams webhook URI'
$Now = Get-Date
$Minutes = '15'
```
Like in the previous script, the variables you need to adjust are these:

**\$ApplicationID** - from the generated application in step 1

**\$ApplicationKey** - from the generated application in step 1

**\$TenantDomain** - your FQDN primary domain

**\$URI** - the generated URI from the creation of the incoming webhook

**\$Minutes** - The last x minutes the script will look for updates. Default in the script is 15 minutes, as I run the script on that schedule.


The script will request new or updated incidents by using the provided application ID and key. It will color code the incident based on state.

- **Red**: Incident
- **Yellow**: Advisory
- **Green**: Incidents with an end time.

As the same incident can show up several times, the script picks the latest index on an incident, to avoid duplicates.

It will then create one or more payloads, and post in the Teams channel.

**4. Test**

Do a dry run to see of the script works as intended with your own variables. If you see no incidents posted, it could be because of the time interval. Try to expand the number of minutes to see if you get any hits.

Once your happy, revert the interval and you're ready to go.

**5. Automation**

To schedule the script, there are several ways. I prefer an Azure automation runbook. As the runbooks can only be run once per hour, you can either create 4 schedules, or set up another [Azure Logic Apps](https://blogs.technet.microsoft.com/stefan_stranger/2017/06/23/azur-logic-apps-schedule-your-runbooks-more-often-than-every-hour/)

Part **III** to follow...