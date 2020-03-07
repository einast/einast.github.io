---
layout: post
title: Daily Pokémon - post to DAKboard
image: /images/DailyPokemon.png
categories: [Powershell, DAKboard, Azure Automation]
---

As the family already use a [DAKboard](https://dakboard.com/site) to keep track of our upcoming tasks and reminders, we often take a peek to see what's happening. To also involve my kids, I thought I'd create something for them.

My kids love Pokémon. I mean really, really love them. Inspired by an addin to another similar type of screen, I'd like to present a random Pokémon every day.

I created a Powershell script that picks a random Pokémon from Pokeapi.co and an image from pokeres.bastionbot.org, pull some stats, format it as JSON and present it on the DAKBoard. My son was the subject matter expert on what to present (also teaching me many things about Pokémon at the same time).

![](/images/DailyPokemon.png)


The script is run on a schedule from an Azure Automation runbook at midnight every day, so the kids can check it out in the morning when they get up.

**High-level**
 • Generate a random number between 1-807 (number of Pokémon in the Pokeapi)
 • Use that number to query the API, pulling data
 • Use the same number to pull an image of the Pokémon
 • Parse the data I would like to present into a JSON file
 • Upload the JSON file to a web server via FTP (using the PSFTP module)
 • Repeat nightly
 • Configure the DAKBoard with the Pokémon section, using the URL of the uploaded JSON payload

The script is pretty self-explanatory with comments.

In the DAKboard portal, configure a new Block, choose **External Data/JSON**.

![](/images/DAKboard001.png)

Put in the URL of the JSON file (remember it has to be https), tick the stats you would like to present. Insert **__image** to refer to the image section in the JSON. Refresh Data is set to the max (4 hours), as we don't need to run it more often (script runs every 24 hours). 

![](/images/DailyPokemonDAK01.png)
![](/images/DailyPokemonDAK02.png)

I shared the script [here](https://github.com/einast/PS_M365_scripts/blob/master/Other%20scripts/DailyPokemon.ps1).
