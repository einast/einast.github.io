---
layout: post
title: Utilizing Azure Automation variables in Runbooks
image: /images/AzAutoTeamsHeader.png
categories: [Powershell, Office 365, Microsoft Teams]
---
I have shared some scripts in my earlier [posts](https://thingsinthe.cloud/Teams-webhook/). However, as I run the scripts in Azure Automation, I made some adjustments in the versions I use in my setup.

![](/images/AzAutoTeamsHeader.png)

In this post, I will share these "Azure Automationized" versions with a focus on how I use variable assets to solve two specific use cases.

Azure Automation is a great way to run recurring scripts in a serverless world. In this post I will not cover how to set up Azure Automation itself, you'll find a lot of info on that elsewhere.

One of the advantages of Azure Automation is the possibility to use variable assets. These are variables you can call from your runbooks, and they are global, meaning you set them in one place, and they can be used by multiple scripts.

If you need to change the value of a variable assets, you do it in one place. This means no change in your runbooks scripts, as they call the same variable name. 

You can also avoid having sensitive data in clear-text in your scripts, for example usernames and passwords. This is a good practice, especially if you use Role-Based Access Control (RBAC) to segregate duties in Azure Automation and want to keep sensitive data visible only for a select set of users. 

More info about RBAC in Azure Automation can be found [here](https://docs.microsoft.com/en-us/azure/automation/automation-role-based-access-control#roles-in-automation-accounts).

I will show how I use variable assets in two scripts, for two different use cases.

*All the scripts I have adapted for Azure Automation, are shared at the end of this post.*

#### Securing your sensitive data

This is a common scenario. You have a set of scripts that contains one or more sensitive variables. Like passwords.

In several of my scripts, I need to use an Application ID and Key. Although these credentials are strictly limited to only read certain data in my tenant, I would still like to avoid having them in clear text in my scripts. 

I use my Application ID and Key for a couple of scripts, since they read from the same data source. By defining them in one place, I can reuse them in several scripts.

The script I will use as a demo is the **O365 Message Center to Teams channel** script. I would like the script to call the variables from Azure Automation, instead of defining them in clear-text in the code.

First I declare the variable assets in my scripts. I name them something logical:

```powershell
$AzVariableApplicationIDName = 'AzVariableMCAppID'
$AzVariableApplicationKeyName = 'AzVariableMCAppKey'
$AzVariableTenantDomainName = 'AzVariableTenantDomain'
$AzVariableTeamsURIName = 'AzVariableMCURI'
```
**Explanation:**

```$AzVariableApplicationIDName``` As I have created an Azure application tailored for reading Office 365 Health data in my previous scripts, I add a variable name belonging to the Application ID here.

```$AzVariableApplicationKeyName``` The Application Key belonging to the Application above. I add a variable name for this. I will use the builtin encryption support when I define it in Azure.
```$AzVariableTenantDomainName``` Create an Azure Automation variable asset name containing your tenant domain FQDN.

```$AzVariableTeamsURIName``` You need to create a Teams webhook variable name where you want your updates to appear.

With that in place I need to create the variable assets with the same names in Azure Automation.


##### Configure Azure Automation variable assets
*(For this post, I will create the variable assets in the azure portal. Normally I use powershell and the [Azure automation ISE Addon module](https://github.com/azureautomation/azure-automation-ise-addon)*.

In the Azure portal, search for **Automation**. Click **Automation account** (if haven't configured one, you need to do that first)

![](/images/AzureAutomation01.png)

Click your automation account

![](/images/AzureAutomation02.png)

Go to **Shared resources**, then **Variables**.

![](/images/AzureAutomation03.png)

Click **Add variable**

![](/images/AzureAutomation04.png)

Beginning with the first declared variable above, fill in the same name. Add an optional Description, select String. Add the value of the variable.
Encryption is optional. Click **Create**

![](/images/AzureAutomation05.png)

Repeat for all the declared variables. 

For the Application Key, I chose to encrypt it, and left the rest of the assets unencrypted. This means that the value will be securely stored in a system defined Key Vault.

Verify that your Azure Automation variable assets now have the same names you declared in the previous step.

I added logic to read out the Azure Automation variable assets I defined in the variable sections of the script, to variables that I use later in the script:

```powershell
# Converting Azure Automation variable assets to script variables
$Tenantdomain = Get-AutomationVariable -Name $AzVariableTenantDomainName
$ApplicationID = Get-AutomationVariable -Name $AzVariableApplicationIDName
$ApplicationKey = Get-AutomationVariable -Name $AzVariableApplicationKeyName
$URI = Get-AutomationVariable -Name $AzVariableTeamsURIName
```
*Please note that in order to read an encrypted variable, you need to use Get-AutomationVariable, and not Get-AzureRmAutomationVariable*

I now have a script ready with variables secured in Azure Automation variable assets. No sensitive data is stored in the script. 

Doing a dry run shows that the script works as intended, the runbook calls the variable assets in Azure Automation, and completes successfully.

![](/images/AzureAutomation06.png)

*You also have variable asset type for credentials (username and password), I did not use these in this script*

#### Using variables for comparison of data - avoid duplicates

I created a script for checking for **Office ProPlus updates to Teams channels** earlier.

Soon after I started using the script in my own tenant, I saw that updates were posted. Great, it worked as intended. 

But the next day, another update was posted. Looking closer, I saw that it had identical content as the previous update.

I checked the page source on Microsoft's web page for the update channel, and I saw that the meta tag I use as a trigger (*"updated_at"*) indeed was changed. So something changed in the page, but it was not a new Office update. Instead I now had duplicates in my Teams channel.

To avoid this, I would like to add some checks to see if there in fact was a new update or not, before posting anything. That meant that I needed to store the last posted update somewhere, so I could do a comparison on consecutive runs of the script.

I outlined a workflow on how I would like to solve this.

![](/images/AzureAutomationWorkflow.png)

 Since I possibly have to create Azure Automation variables that does not exist, I will use cmdlets from the **AzureRM.Automation module**.
 
 I need to declare some new variables in the script. I add the necessary variables for the parameters I will run in the script:

```powershell
# Azure automation specific variables
$AzAutomationAccountNameVariable = 'AzVariableAutomationAccount'
$AzAutomationResourceGroupVariable = 'AzVariableResourceGroup'

# Monthly channel 
$AzAutomationURIMonthlyVariable = 'Name of your Automation Teams Monthly URI Variable' # Comment out to _not_ check this channel
$AzAutomationPayloadMonthlyVariable = 'Name of your Automation Monthly Payload Variable' # Will be created by the script if not existing

# SACT channel
$AzAutomationURISactVariable = 'Name of your Automation Teams Semi-Annual Channel (targeted) URI Variable' # Comment out to _not_ check this channel
$AzAutomationPayloadSACTVariable = 'Name of your Automation Semi-annual (targeted) Payload Variable' # Will be created by the script if not existing

# SAC channel
$AzAutomationURISacVariable = 'Name of your Automation Teams Semi-Annual Channel URI Variable' # Comment out to _not_ check this channel
$AzAutomationPayloadSACVariable = 'Name of your Automation Semi-Annual Payload Variable' # Will be created by the script if not existing
```
**Explanation:**

The first two I need since I use *Get-AzureRmAutomationVariable* and not *Get-AutomationVariable*. The reason being some issues with reading out the values.
```$AzAutomationAccountNameVariable``` Name of your Automation Account Name Variable

```$AzAutomationResourceGroupVariable``` Name of your Automation Resource Group Variable

For each of the Office ProPlus channels, I created two variables. Repeat for every channel:
```$AzAutomationURIMonthlyVariable``` Name of your Automation Teams Monthly URI Variable. Comment out to **not** check this update channel

```$AzAutomationPayloadMonthlyVariable``` **This will be created by the script if not existing.** If you want to, you can create it. Give it a name matching your Automation *Monthly/SACT/SAC* Payload Variables.

In addition, I added some logic to see if the variables exist if not they will be created. Also a check to see of the content of the new payload differs from the "old" :

```powershell
If (!(Get-AzureRmAutomationVariable -AutomationAccountName $AzAutomationAccountName -Name $AzAutomationPayloadMonthlyVariable -ResourceGroupName $AzResourceGroup -ErrorAction SilentlyContinue)) {
      New-AzureRMAutomationVariable -AutomationAccountName $AzAutomationAccountName -Name $AzAutomationPayloadMonthlyVariable -ResourceGroupName $AzResourceGroup -Value $null -Encrypted $false
    }
    Else {    
    }

.......

If ($payload -ne $payload.stored.in.Azure) {
    Invoke-RestMethod -uri $URI -Method Post -body $payload -ContentType 'application/json; charset=utf-8'
    Set-AzureRMAutomationVariable -AutomationAccountName $AzAutomationAccountName -Name $AzAutomationPayloadVariable -ResourceGroupName $AzResourceGroup -Value $payload.stored.in.Azure -Encrypted $false  
    }
```

In Azure Automation, I create the necessary variable assets for the Automation parts and the Office ProPlus update channels, using the same name as I declared in the script. As mentioned, I did not create the payload variables, since the script will create them for me if they don't exist.

[See the previous section on how to create the variable assets in Azure Automation](#Configure-Azure-Automation-variable-assets)

For these variable assets, I did not use encryption.

With everything in place, I do a dry run (I altered the time frame to match the *updated_at* meta tag in the webpage, in order to ensure that the script catches some new updates).

I see new updates for all the Office Proplus update channels.

![](/images/AzureAutomation07.png)

Then I take a look in Azure Automation, and I see that the script created the variable assets with the content from each Office ProPlus channel:

![](/images/AzureAutomation09.png)

Looking at the value of one of the Payload assets, I see that the script populated it with the latest payload:

![](/images/AzureAutomation10.png)

This is the payload that consecutive script executions will check against. If no differences, nothing will be posted. But if there are changes, the new payload will be posted to Teams, and the new payload will overwrite the old Azure Automation variable asset.

I do another dry run (which will find the same updates due to me tampering with the time frame to check in). Since the script finds no new unique content when comparing the payload with the previous payload (now stored as Azure Automation variable assets) it does not post anything.

Lastly I change the timeframe back to my desired time window to look for updates, and schedule the script to run.

This shows that the Azure Automation variable assets are useful for different scenarios, not just to store credentials.

The new Azure Automation runbooks scripts can be found [here](https://github.com/einast/PS_M365_scripts/AzureAutomation).