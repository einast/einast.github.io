---
layout: post
title: Teams Message Cards - Part I (Microsoft 365 Roadmap)
image: /images/TeamsRoadmapWebHook3.PNG
categories: [Powershell, Office 365, Microsoft Teams]
---

This is part I of a series where the goal is to both create a reusable template as well as be able to present useful information from the [Microsoft 365 Roadmap](https://www.microsoft.com/en-us/microsoft-365/roadmap) in a Teams channel.

Tools used:
- Powershell
- [Message Card playground](https://messagecardplayground.azurewebsites.net/)

As the roadmap URL provides an RSS feed, I can use Powershell to:
- Get data (I would like to do this once per day)
- Parse the data, select the parts I want
- Adapt the output as JSON in a supported Message Card format
- Post it in the Teams channel by using an incoming webhook

Further, I would like the message cards to be color coded based on the status of the feature:
- Red: In development
- Yellow: Rolling out
- Green: Launched

This resulted in a short script to show any updates from the roadmap in the last 24 hours.

![](/images/TeamsRoadmapWebHook3.PNG)


#### Setup ####

- Set up an [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/connectors/connectors-using) in your selected Teams channel
- Copy the URI that the connector generated (you need that later)
- Download the script from [here](https://github.com/einast/PS_M365_scripts/blob/master/M365RoadmapUpdates.ps1)
- Adjust the user defined variables to fit your environment, as mentioned above
- Do a dry-run to see if anything is posted. If not, try to adjust the **\$Hours** variable to a higher value

#### Walkthrough ####
The script is pretty basic. All that is needed, is to adjust the **\$URI** and **\$Hours** variables.

```powershell
# User defined variables
$URI = 'Teams webhook URI'
$Roadmap = 'https://www.microsoft.com/en-us/microsoft-365/RoadmapFeatureRSS'
$Hours = '24'
$Now = Get-Date 
```

Please note that **\$Now** variable is needed to calculate the time, so don't change that.

To request the data, it runs an Invoke-RestMethod with the variables defined.

```powershell
# Request data
$messages = (Invoke-RestMethod -Uri $Roadmap -Headers $headerParams -Method Get)
```

The script then:
- Goes through any new announced features and calculates if they happened within the defined timeframe (last 24 hours)
- Further it does a count of the message.category values, as we need to separate the first value from the rest
- In order to avoid a failed JSON payload, we need to convert the string to JSON before we build the payload
- Then the script will color code each entry based on state of the roadmap item

```powershell
# Parse data
ForEach ($msg in $messages){

        # Add updates posted last 24 hours                
        If (($Now - [datetime]$msg.pubDate).TotalHours -le $Hours) {
                
                # Count, join and prepare category for use in the card
                $categoryno = $msg.category.Count
                $category = $msg.category[1..$categoryno] -join ", "
                
                # Convert MessageText to JSON beforehand, if not the payload will fail.
                $Message = ConvertTo-Json $msg.description

                #Set the color line of the card according to the Status of the environment
                if ($msg.category[0] -eq "In development")
                    {
                    $color = "ff0000"
                    }
                    else
                        {
                            if ($msg.category[0] -eq "Rolling out")
                                {
                                    $color = "ffff00"
                                    }
                                    else
                                        {
                                            $color = "00cc00"
                                            }
                                 }
```

Then we use our data to create a JSON payload. I choose to create it with a here-string, adding the values and setting the color.

The [message card playground](https://messagecardplayground.azurewebsites.net/) is a good place to validate your JSON payload.

```powershell   
# Generate payload(s)          
$Payload =  @"
{
    "@context": "https://schema.org/extensions",
    "@type": "MessageCard",
    "potentialAction": [
            {
            "@type": "OpenUri",
            "name": "More info",
            "targets": [
                {
                    "os": "default",
                    "uri": "$($msg.Link)"
                }
            ]
        },
     ],
    "sections": [
        {
            "facts": [
                {
                    "name": "Status:",
                    "value": "$($msg.category[0])"
                },
                {
                    "name": "Category:",
                    "value": "$($category)"
                }
            ],
            "text": $($message)
        }
    ],
    "summary": "$($msg.Title)",
    "themeColor": "$($color)",
    "title": "Feature ID: $($msg.guid.'#text') - $($msg.Title)"
}
"@
```

Lastly, *if* there are any new updates, post them in the defined Teams channel:

```powershell
# If any new posts, add to Teams
Invoke-RestMethod -uri $uri -Method Post -body $Payload -ContentType 'application/json; charset=utf-8'
```

- When you're happy, schedule the script to run on a schedule that aligns with your **\$Hours** variable:
    - Being cloud-first, Azure automation is what I use

This script is the base for the next scripts.

Part **II** is available [here](https://thingsinthe.cloud/Teams-message-cards-Office-365-Health-status/).
