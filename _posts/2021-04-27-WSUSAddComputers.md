---
layout: post
title: WSUS - script to add computers from different domains and workgroups, and handle Defender updates
image: /images/iu.jpeg
categories: [Windows, WSUS, Powershell]
---

A new customer case, this time with an on-premises data center and server patching. They currently don´t have (or want) any SCCM in production, and required a method for handling only approved patches. So WSUS was the selected tool. Since this customer have a few different Active Directory domains, as well as computers in workgroups, I needed to get them into WSUS using registry entries. In this case, they had approximately 60 servers to patch.

Requirements:
- No automatic installation of patches, should be handled in maintenance windows manually (Only download and notify)
- Fine-grained control of which patches to apply (WSUS approved only)
- Two defined computer groups to apply patches to, one for pilot and one for broad deployment

One of the issues is Microsoft Defender updates. With WSUS and no automatic patching, these would not be installed as often as required. So this needed a solution. One possible workaround is to automatically approve updates that does not require a restart, but that could potentially apply to other updates as well. So we dropped that option.

To avoid clicking around in each server, I wrote a quick script that the customer could run on their servers, regardless of domain or workgroup.

This script does the following:
1. Prompts which WSUS computer group the computer should be a part of
2. Sets the registry according to the customer´s requirements and the WSUS computer group based on the answer above
3. Checks for C:\Scripts, if not, creates the directory
4. Creates a new script in C:\Scripts called DefenderUpdate.ps1 with the command to update Defender using MicrosoftUpdateServer
5. Creates a scheduled task to run the DefenderUpdate.ps1 every 6 hours (easily adjustable)
6. Try to force computers to check in to the WSUS server (mixed success) to avoid the long delay of them showing up in WSUS

The script is a starting point, and can be developed to fit other customer cases. It´s found [here](https://github.com/einast/PS_M365_scripts/blob/master/WSUSAddComputers.ps1).