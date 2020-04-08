---
layout: post
title: Office 365 Service Health - push notifications
image: /images/PushoverHeader.png
categories: [Office 365, Microsoft Teams, Azure Automation, Pushover]
---

In these Covid-19 times, Office 365 (soon to be Microsoft 365) is becoming more and more important (and popular).

The health script I wrote a while back is a great way to keep track of any issues that arise in Office 365 (you'll find the post [here](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/). And ufortunately, there have been a lot of issues since everybody started working from home. Microsoft even altered their services to keep up with the increased usage load.

However, as the script posts to Microsoft Teams, what happens if Teams struggle, so the script is unable to post to the Teams channel? Or if the script is unable to run?

With Business Continuity Planning (BCP) in mind, I decided to at least add an option for the script to also send push notifications. I wanted to use a service outside of Microsoft. I have used [Pushover](https://pushover.net) earlier, and went with that service here as well.

Pushover is a service with a low cost, and can be tailored to anything from individuals to teams.

Start off with registering (or log in if you already use Pushover).

In Pushover, click **Apps & Plugins**, and then **Create a New Application / API Token**.

![](/images/Pushover01.png)

Fill in **Name**, a **Description**, **URL** (optional) and upload an **Icon**. Click **Create Application**.

Click you new application.

![](/images/Pushover02.png)

Copy your **API Token/Key** to your clipboard, we'll need that.

Since my scripts are adapted to Azure Automation, I add these tokens as variables there.

Go to your **Azure Automation account**, and add a new variable. This variable will contain your **API Token/Key**. I choose to encrypt it.

![](/images/Pushover04.png)

In addition we also need the user token. You can find it in **Settings** in Pushover, under **Reset User Key** (you don't need to reset it, it's only if you suspect your key has been compromised). Copy to value to your clipboard.

![](/images/Pushover03.png)

Repeat the step above, by creating an Azure Automation variable for your user key as well. **This key can also be a group key if you want to notify a group**.

Now we are ready to use this. Grab the latest version of the script from -----

Adapt the variables section to your environment.

The section you need to change, is the **Pushover notification** part. Comment out the entire section if you don't want to use Pushover.

Change *PushoverToken* and *PushoverUser* to match your Azure Automation Variables. Set *Pushover* to **yes** or some other value. I use yes for readability.

```Powershell
# ------------------------------------- USER DEFINED VARIABLES -------------------------------------

$ApplicationID = Get-AutomationVariable -Name 'AzApplicationID'
$ApplicationKey = Get-AutomationVariable -Name 'AzApplicationKey'
$TenantDomain = Get-AutomationVariable -Name 'AzO365TenantDomain'
$URI = Get-AutomationVariable -Name 'AzAzureHealthURI'
$Minutes = '15'

# Pushover notifications in case Teams is down.
# Due to limitations and readability, the script will only send the title of the incident/advisory to Pushover. 
# COMMENT OUT THIS SECTION IF YOU DON'T WANT TO USE PUSHOVER!

$Pushover = 'yes' # Comment out if you don't want to use Pushover. I use 'yes' for readability.
$PushoverToken = Get-AutomationVariable -Name 'AzO365PushoverToken' # Your API token. Comment out if you don't want to use Pushover
$PushoverUser = Get-AutomationVariable -Name 'AzO365PushoverUser' # User/Group token. Comment out if you don't want to use Pushover
$PushoverURI = 'https://api.pushover.net/1/messages.json' # Default Pushover URI. Comment out if you don't want to use Pushover
```

Schedule the script, and it will run periodically (I run it every 15 minutes).

The script will now both post to Teams and send a push notification through Pushover.

![](/images/Pushover04.jpg)

Two caveats:
- I did not add logic to keep within the Pushover message limits. So some long messages might be truncated. But for now, the most important part was to enable push notifications through a non-Microsoft service.

- Noise (potentially many push messages). I did not add logic to separate between incidents and advisories for push notifications. For now, all messages going to Teams will also be sent to Pushover. One idea is to limit push notifications to Teams only incidents and/or advisories, I might look into that later.