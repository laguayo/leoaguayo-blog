---
layout: post
title: "BRZ Scraper"
date: 2013-05-23 14:12
comments: true
categories: ['brz', 'subaru', 'cars', 'code', 'python', 'app engine']
published: false
---
<div>I recently bought a new car.{% emoji smile %} But that’s not that awesome many people buy cars everyday, what makes this occasion special you ask? I bought a first year model made in collaboration by two amazing manufacturers, Toyota and Subaru. </div>

{% img /images/brz-render-passenger.png BRZ DGM Render %}

<div>It is the Subaru BRZ. Also known as Scion FRS or Toyota GT-86 in Japan. Still not that great car just came out who cares... </div>

Well I started seeing this car a few months ago driving by on my commute to work more and more so naturally I became curious. I did my research and when I learned the backstory I was completely enthused. The opportuntity arrived that me and my significant other would need a new car, how convenient right? We start car shopping one thing led to another and we decided on getting the BRZ. 

I started to shop around trying to find a good deal and learned this is a hard car to acquire at a fair price. Since it is it's first official year and it's so rare most dealers are marking up prices, or have no inventory.

Hmm what to do...I started to browse the internet and I noticed most Subaru dealers have the same site layout...Actually all individual Subaru dealers use the same layout.

{% img center /images/subaru-dealer.png 485 300 Subaru Dealer %}

So my engineering mind went into hyperactive mode. I planned it out, I would just scrape all the Subaru dealers nearby or even in the same state and index all their inventory then just call up those that met my criteria and I would have my new car.

Also conveniently I wanted an excuse to learn a new language, a fellow colleague once commented, “if you want to scrape a site use python, it’s really easy”...So I did, I've never used python before so I decided why not learn python and find a new car all in one go and I did. This is where the coding starts, if you don't care skip ahead to the ending.

<!-- more -->

<h2>Day 1: Simple scrape, BeautifulSoup and CLI</h2>

I was surprised that I was able to learn basic python and the scraping library <a href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup</a> enough to put a script together in the span of 3 hours. 
``` python
#!/usr/bin/python
import urllib2
import re
from bs4 import BeautifulSoup

def printCars( mylist ):
	for row in mylist:
		for string in row.stripped_strings:
				if '2013 Subaru BRZ' in string:
						print('\n' + string)
						if "Limit" in string:
								print("+++++++++")
				elif string == 'Satin White Pearl':
						print('\tSatin White Pearl<============WHITE')	
				elif string == '6-Speed Automatic':
						print('\t6-Speed Automatic<============AUTOMATIC')	
				elif '$' in string:
						print('\t' + string)						
				elif string not in ('VIN:','More','...',':','Adjusted Price', 'Get ePrice', 'Engine:', 'H-4 cyl', ',', 'Drive Line:','RWD', 'Transmission:', 'Interior Color:', 'Dark Gray', 'Model Code:', 'Details', '2.9% Financing','Watch Video', 'Compare','Compare Selected', 'View Details', 'Manufacturer Offer:'):
						print('\t' + string)

	return
	
def getCars( site ):
	print '\n\nOPENING :' + site
	page = urllib2.urlopen(site).read()
	soup = BeautifulSoup(page)
	cars = soup.find_all("li", class_="inv-type-new")
	printCars(cars)
	return
			
pages = ['http://www.stevenscreeksubaru.com', 'http://www.livermoresubaru.com',
		 'http://www.subaruofoakland.com', 'http://www.carlsensubaru.com', 
		 'http://www.marinsubaru.net', 'http://www.albanysubaru.com']
inventoryString = '/new-inventory/index.htm'
modelParam = '?model=BRZ&'

for subsite in pages:
	site = subsite + inventoryString + modelParam
	getCars(site)
```

Output like this
``` clear
Leos-MacBook-Pro-2:scraper leoaguayo$ ./scrapingWeather.py

OPENING :http://www.stevenscreeksubaru.com/new-inventory/index.htm?model=BRZ&

2013 Subaru BRZ Premium
	$33,979
	6-Speed Automatic<============AUTOMATIC
	Exterior Color:
	Sterling Silver
	DZB
	JF1ZCAB12D2609827

2013 Subaru BRZ Premium
	$32,078
	6-Speed Manual
	Exterior Color:
	Satin White Pearl<============WHITE
	DZA
	JF1ZCAB17D1609912
	…

OPENING :http://www.livermoresubaru.com/new-inventory/index.htm?model=BRZ&

2013 Subaru BRZ Limited
+++++++++
	MSRP
	$28,265
	6-Speed Manual
	Exterior Color:
	Sterling Silver
	DZE
	JF1ZCAC1XD1610017
	…

```
Some real python developers might cringe at the thought of this code and comments are appreciated as I am not versed in python best practices but I did my best in the short time I spent.

So now I have a basic command line script I can run that will show me all the dealers with cars in their online inventory, useful to run a few times a day to see if I catch any fish but then I thought. What I could run this every so many minutes on a timer and then collect the data and send myself a notification, would be more useful. 

Then I had a second thought this would be easier as a service or a running webapp independent of my local computer.  So the first thought I had, being an Java App Engine Developer, it would be easiest for me to port my python scrape to a python App Engine instance, that was the start of day 2.

Day 2:Re-Learned App Engine, Webapp2, jinja2 contemplated Django

App Engine in Python was a completely massive task, especially since I was working on this project on my free time after and before work. I not only learned App Engine’s Python quirks I also gained a deeper understanding of python and its modules, classes, list, sets and dictionary data structures. I also re-factored most of my scraper code. Yet with all this learning curve I was able to pull something together fairly quickly, in one night I had a basic list of cars showing up on an index page loaded from the database. I ran the script manually because I wasn’t sure how far I could take the limits of a free app engine instance, especially once I encountered bugs and task would repeat endlessly until my quota ran out. 

Day 3: Put it all together this is what we have.

After day 3 I had a list of dealers, a list of cars and some basic bootstrap and a few days later I added a js data table that could sort and filter the list of cars. Through the span of that week I would tweak little things here and there. I managed to get my task up and running.

{Pic of brz-scraps}

The End...After running the program for a week I started calling dealerships and even with all the code in the world, it wasn't enough. Some dealers would advertise MSRP online and then when you called or came in there would be a $5000 dollar markup sticker next to the car. So I let the program sit. I finally made a few plans to go down to Southern California to pick up my car in a few weeks once some dealers got their shipments in from Subaru.

Monday Day, after giving up for another 2 weeks, I was looking over my app engine logs and I noticed my script would hit a bug on one of the latest dealers I had added to the list, I fixed the bug re-deploy and boom their it was the car I was looking for sitting in a local dealer nearby. I called the dealer immediately and within the next few hours I was signing my life away.

Day 5 Presentable tweaks

After acquiring my car I knew I wanted to tell the world about my story so I added more user admin features to limit people who could access and edit parts of the site in case others want to add more dealers to the scraping script.

Github Page https://github.com/laguayo/brz-scraps
App Engine Link: brz-scraps.appspot.com
