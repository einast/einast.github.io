---
layout: post
title: Offboarding a large number of users from Microsoft 365
image: /images/offboarding.png
categories: [Powershell, Microsoft 365, Azure]
---

Quite some time since the last post here, as I have been busy with changing jobs and a certification "marathon".

A customer asked me if I could help them offboard a large amount of users in Microsoft 365 simultaneously, due to an organizational change. This had to happen relatively fast, so they wanted to know if it was possible to automate the task.

I started off with a script that does the following:

- Block user sign-in and disconnect any active sessions
- Convert the user mailboxes to shared mailboxes
- Sets an auto-reply on their mailbox
- Remove the users from any distribution lists
- Cancel any meetings the user had organized
- Add a new (external) contact object for the users with an external e-mail address defined in the CSV (for other internal users to reach them if needed)
- Remove all assigned Microsoft 365 licenses
- I will probably add more functionality as we move along...

Since time was of the essence, I decided to use a CSV file with the necessary data, parsing it and cycle through each affected user.

I created a script for this task, with only one variable that the customer needed to change, which is the CSV file.

The CSV file has three columns:
Name,internal.email,external.email

Where:

- **Name** is the user given name
- **internal.email** is the UPN of the user
- **external.email** is the external e-mail address that we want to create a contact object for

I have shared the script [here](https://github.com/einast/PS_M365_scripts/blob/master/M365Offboarding.ps1). The script can be developed further, like removing the user from Office 365 groups etc.