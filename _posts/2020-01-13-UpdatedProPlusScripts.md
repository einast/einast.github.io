---
layout: post
title: New versions of the Office 365 ProPlus updates Teams scripts
image: /images/O365_Monthly_updates_(Office365_Test)_Microsoft_Teams.png
categories: [Powershell, Office 365, Microsoft Teams, Azure Automation]
---

After the first Office ProPlus updates for 2020 showed up last week, I saw jibberish as output in the Teams channels. Looking into the Microsoft websites for the different update channels, I saw that Microsoft added an additional **Note** section to all channel changelog pages, as opposed to 2019:

![](/images/ReleaseNotes.png)

This caused parsing of the wrong section to the payload. This is now mitigated by changing the regex in the scripts. Testing today, I see that the new regex syntax parses the correct output, and post the right section to Teams:

![](/images/ProPlusScriptMitigated.png)

Scripts updated:
- [Latest Azure Automation version I use](https://github.com/einast/PS_M365_scripts/blob/master/AzureAutomation/AzOfficeProPlusUpdates_(without_runas_account).ps1)
- [My first Azure Automation version](https://github.com/einast/PS_M365_scripts/blob/master/OfficeProPlusupdates.ps1)
- [The first generic version](https://github.com/einast/PS_M365_scripts/blob/master/AzureAutomation/AzOfficeProPlusUpdates.ps1)