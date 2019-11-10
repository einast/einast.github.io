---
layout: post
title: Office ProPlus updates - to Teams channels
image: /images/O365_Monthly_updates_(Office365_Test)_Microsoft_Teams-preview.png
categories: [Powershell, Office 365, Microsoft Teams, OfficeProPlus]
---

At customers, we normally have users split on the different Office ProPlus update channels. A few users on the Monthly channel, some more on the Semi-Annual Channel (targeted), and the major set of users on the Semi-Annual Channel. This is to be able to test new versions and features in a controlled matter. 

In order to get information about updates for these Office ProPlus channels, we don't want to be dependent on checking Microsoft's website. Instead we would like to get the same information posted into one or more Microsoft Teams channels as soon as there are any new updates available.

#### High-level ####
I will re-use the code from my previous posts [Teams Message Cards - Posting useful information](https://thingsinthe.cloud/Teams-webhook/) to facilitate this.

As many use more or all of the Office ProPlus Monthly Channel, Semi-Annual Channel (Targeted) and Semi-Annual Channel, there are quite a few updates to keep track on.

Unfortunately Microsoft does not provide an RSS for these updates, meaning that we need to browse their site to get the latest updates.

In this use case, I would like to get the latest update from for example ["Release notes for Monthly Channel releases in 2019"](https://docs.microsoft.com/en-us/officeupdates/monthly-channel-2019), and post them in a Teams channel.

Screenshot of what I want (latest entry only):
![](/images/Monthly.PNG)

However, looking at the page, all entries from the current year are listed chronologically, while I only want the latest one at the top:
![](/images/Monthly02.PNG)

I took a look at the page source to see if there were any tags I could hook in to. I needed:
- Some form of date when the page was last updated
- The header of the latest post
- A way to split the posts apart, giving me the possibility to only grab the latest entry.

The changelog pages for all Office ProPlus Channel are all created in the same way.

For the first requirement, I spotted a timestamp tag:
```html
<meta name="updated_at" content="2019-11-01 08:22 PM" />
```
Great! This one can be used for time calculation.

Further I needed to find a pattern for the update posts. Looking further into the code, I found that all posts from the newest to the latest, are all between two **\<h2>** tags. Also I see the header I want within the **\<h2>** tags:

```html
<h2 id="october-15">October 15</h2>
<h3 id="non-security-updates-1">Non-security updates</h3>
<h3 id="office-suite-2">Office Suite</h3>
<ul>
<li>We have temporarily disabled the Cloud Save dialog to address the saving issue we posted on October 14, 2019 for builds older than 12130.20184. This feature will be re-enabled soon.</li>
</ul>
<h2 id="version-1909-october-14">Version 1909: October 14</h2>
```

So basically I need all data starting at the first **\<h2>** tag, and all the way until right before the next **\<h2>** tag.

Earlier I have done something similar for another web page, where I needed to list out my data quota at a mobile broadband provider. That was a couple of years back, where I used *Invoke-WebRequest*.

I started by trying to use the same cmdlet, however, it did not work. Running the command against the Microsoft page just froze. Digging into that, I quickly found out that others experience the same. Also starting from Powershell 6.0, *Invoke-WebRequest* only support Basic parsing. Which breaks the previous method, where I parsed text from InnerHTML, not available with Basic parsing.

To overcome the problem, I used *Invoke-RestMethod*. This command is better suited for XML and JSON parsing, but it works fast and I was able to read the page into a variable.

#### Timestamp ####
Continuing, I first needed to parse just the timestamp into a separate variable. Since the web page is pretty basic, I choose to use *regex*.

There are many tools available that are helpful for regex testing. This time I used [Regexr](https://regexr.com/).
![](/images/regexr01.PNG)

I will not go into how to build regular expressions, both because I'm by far any regex expert, and also because there are many guides out there for the same.

After parsing the webpage, and building the regular expression for the timestamp, I ended up with the following pattern:
```
\d{4}-\d{2}-\d{2} \d{2}:\d{2} [AP]M
```
This will grab the update timestamp from the webpage.

#### Latest update - heading ####
I would like to give the cards title based on the title of the update. In regexr.com, I built this expression:
```
(?<=\<h2.*?\>)(.*?)(?=<\/h2\>)
```

#### Latest update - content ####
Again using regexr.com, I built the following expression for the latest posted update:
```
(\<h2.+?\>)((.|\n)+?(?=<h2.+?\>))
```
This grabs the text starting with the first **\<h2>** tag and ending before the second **\<h2>**.

In the script, I use the patterns above to get the data I want to be presented in one (or more) Teams channels. The rest is basically reusing the scripts created in the previous posts.

#### The script ####
What is needed, is to set the user variables:
```powershell
# User defined variables
# ----------------------
# If you want to check Monthly Channel, Semi-Annual Channel Targeted (SACT) and/or Semi-Annual Channel, add your Teams URI in the variables fields. 
# Leave the ones you don't want to check blank.
 
$MonthlyURI = 'https://teams.uri.url.for.monthly.channel/blahblah'
$SACTURI = ''
$SACURI = ''
$Hours = '12'
$Color = '00ff00' # Green
```
*\$MonthlyURI*, *\$SACTURI* and *\$SACURI* are variables for Teams webhook URIs. Either creat new webhooks in Teams, or reuse any if you want.

In my POC, I used the same Teams URI for all Office ProPlus channels, resulting in all updates coming into the same Teams channel. Adapt as you want. Keep the ones you don't need/want blank ( '' ), and the script will skip checking those update channels.

I used 12 hours in my POC, since I ran the script at that frequency. The script then checks for any updates on the webpage the past 12 hours. Change to the number of hours that fits your setup, and run the script accordingly.

I set the color of my message cards to green, but if you want any other color scheme, change it to what you like.

Running the script will provide output similar to this:
![](/images/O365_Monthly_updates_(Office365_Test)_Microsoft_Teams.png)

As usual, I run the script using Azure Automation runbooks.

You find the script here: [OfficeProPlusupdates.ps1](https://github.com/einast/PS_M365_scripts/blob/master/OfficeProPlusupdates.ps1)

##### Disclaimer: The script is tested and working at the moment. If Microsoft changes the layout of their pages, things might stop working. ####