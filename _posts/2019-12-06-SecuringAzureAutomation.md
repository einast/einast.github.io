---
layout: post
title: Securing the Azure Automation runbook for the Office 365 ProPlus updates Teams script
image: /images/SecuringAzureAutomation.png
categories: [Powershell, Office 365, Microsoft Teams, Azure Automation]
---

A short post about a new use case. I was asked to implement my ProPlus channel update script in a PROD environment. I was provided with an
Azure Automation account. Looking at it, I saw that it did not have a Run As account. The customer did not want to allow an Azure Automation Run As account, as it per default gets Contributor role to the entire subscription. It can be scoped down with RBAC, but a decision was made to try to avoid it altogether. Fair enough, I had to take another look at my script.

So I needed to rewrite my code to comply with the security requirements. I used the initial script as a starting point.

Then I made a few changes:

- Replaced some of the used cmdlets with ones that did not require a Run As account, also following the best practice [recommended](https://docs.microsoft.com/en-us/azure/automation/shared-resources/variables#using-a-variable-in-a-runbook-or-dsc-configuration) by Microsoft.

- Manually create the payload asset variables (in encrypted form) before running the script (as opposed to earlier, where they script created these assets if not already there).

- Recreated all other variable assets as encrypted.

Some minor tweaks was needed as well, but I now have a working runbook that is more secure than earlier.

- It's working without a Run As account
- All variable assets encrypted

I have kept the previous version I created, and instead added a new script [here](https://github.com/einast/PS_M365_scripts/blob/master/AzureAutomation/AzOfficeProPlusUpdates_(without_runas_account).ps1).
