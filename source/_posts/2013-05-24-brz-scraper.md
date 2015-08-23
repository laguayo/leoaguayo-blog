---
layout: post
title: "BRZ Scraper"
date: 2013-05-24 14:12
comments: true
categories: ['brz', 'subaru', 'cars', 'code', 'python', 'app engine']
published: false
---
<div>
	I recently bought a new car. {% emoji smile %} I bought a first year model made in collaboration by two manufacturers, Toyota and Subaru. It's the Subaru BRZ, also known as Scion FR-S or Toyota 86 in Japan.
</div>
<div>
{% img /images/brz-render-passenger.png BRZ DGM Render %}
</div>

It's special because it drove me to learn python and then I saw it as a rite of passage as a developer that I would write software to find my dream car.

A few months ago I saw this car driving by on my commute to work, once with Scion badges and then with Subaru badges so I became curious. I did my research and when I learned the backstory, twin cars made by two different companies, I was drawn. The opportuntity arrived that my significant other and I would need a new car, how convenient right? We start car shopping one thing led to another and we decided on getting the BRZ. 

I started to shop around trying to find a good deal and learned this is a hard car to acquire at a fair price. Since it is it's first official year and it's so rare most dealers are marking up prices, or have no inventory.

Hmm what to do...I started to browse the internet and I noticed most Subaru dealers have the same site layout...Actually all individual Subaru dealers use the same layout.

{% img center /images/subaru-dealer.png 485 300 Subaru Dealer %}

I had a plan, I would just scrape all the Subaru dealers nearby or even in the same state and index all their inventory then just call up those that met my criteria and I would have my new car.

Also conveniently I wanted an excuse to learn a new language, a fellow colleague once commented, {% blockquote %} if you want to scrape a site use python, it’s really easy{% endblockquote... %} I've never used python before so I decided why not learn python and find a new car all in one go and I did.

<!-- more -->

<h2 id="Day1">Day 1: Simple scrape, BeautifulSoup and CLI</h2>

I was surprised that I was able to learn basic python and the scraping library <a href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup</a> enough to put a script together in the span of 3 hours. 

Some real python developers might cringe at the thought of this code and comments are appreciated as I am not versed in python best practices but I did my best in the short time I spent.

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
				elif string not in ('VIN:','More','...',':','Adjusted Price', 
				'Get ePrice', 'Engine:', 'H-4 cyl', ',', 
				'Drive Line:','RWD', 'Transmission:', 
				'Interior Color:', 'Dark Gray', 'Model Code:', 
				'Details', '2.9% Financing','Watch Video', 
				'Compare','Compare Selected', 
				'View Details', 'Manufacturer Offer:'):
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
{% highlight stripall %}
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
{% endhighlight %}

I had a basic command line script I can run that will show me all the dealers with cars in their online inventory, useful to run a few times a day to see current inventory. Then I thought I should run this every so many minutes on a timer and then collect the data and send myself a notification, would be more useful. 

Then I had a second thought this would be easier as a service or a running webapp independent of my local computer. Being an Java App Engine Developer, it would be easiest for me to port my python script to a python App Engine instance, that was the start of day 2.

<h2 id="Day2">Day 2: Re-Learned App Engine, Webapp2, Jinja2 contemplated Django</h2>

App Engine in Python was a completely massive task, especially since I was working on this project on my free time after and before work. I not only learned App Engine’s Python quirks I also gained a deeper understanding of python and its modules, classes, <a href="http://docs.python.org/2/tutorial/datastructures.html">list, sets and dictionary data structures.</a> 

I also learned about app engine framework webapp2 and how makes it easy to put together restful application fairly quickly. I then turned to use jinja2 as a quick markup template parser for generating content on the fly. Lastly I contemplated Django but I felt the learning curve was too steep, maybe I was wrong.

I planned out that I would have a 3 main classes, main.py would host all page navigation between the car list to the dealer list and editing each individual dealer. I would put all my data processing in my DataProcesses.py and last I would define all my entities in Entities.py so all schema changes were declared in one place. 

I also made one parent layout.html page for jinja to inherit a general layout and then redfine the body content in each individual page. I played with bootstrap and data tables js to put the view together for each page.

I also re-factored most of my scraper code. Yet with all this learning curve I was able to pull something together fairly quickly, in one night I had a basic list of cars showing up on an index page loaded from the datastore. I ran the script manually because I didn't want to hit a bug running in task and potentially going over quota.

After this I began using Git Version Control. Most of this is documented in the <a href="https://github.com/laguayo/brz-scraps/">github repo</a>.

You can browse the code from the <a href="https://github.com/laguayo/brz-scraps/tree/0df03b9e49ce08fb7b25912a8f592ef382164ffb">initial commit</a> on github, it's not very pretty I made more robust changes in the later commits.


<h2 id="Day3">Day 3: Put it all together this is what we have.</h2>

After day 3 I had a list of dealers, a list of cars and some basic bootstrap and a few days later I added a js data table that could sort and filter the list of cars. Through the span of that week I would tweak little things here and there. I managed to get my task up and running.

A list of notable things
<h4>I redid my parser after much debugging.</h4>
``` python
def parse_car(self, car_str, url):
		try:
			vin = unicode(car_str.find(class_='hproduct').attrs['data-vin'])
			c = Car(key_name=vin, vin=vin)
			check_car = Car.get(c.key())
			if check_car:
				logging.info("Car already exist in db: %s", c.vin)
				return c.key()
			media = car_str.find(class_='media')
			c.model = unicode(car_str.find(class_='url').string)
			c.price = self.get_price(car_str.find_all(class_='value'))
			c.link = unicode(media.a.attrs['href'])
			c.img_src = unicode(media.img.attrs['src'])
			try:
				desc = car_str.find(class_='description')
				desc_dict = dict()
				# desc_dict = dict(e.split(':') for e in description.split(','))
				k = ''
				for s in desc.dl.stripped_strings:
					if (',' not in s) and ('More' not in s) and (u'\u2026' not in s):
						if ':' in s:
							k = s.replace(':','')
						else:
							desc_dict[k] = s
			except (ValueError, TypeError):
				logging.error('Error Parsing Car line Description %s ', 
					car_str.find(class_='description'))
				return
			logging.info('Car Description dict %s', desc_dict)
			if 'Transmission' in desc_dict:
				c.transmission = unicode(desc_dict['Transmission'])
			if 'Exterior Color' in desc_dict:
				c.ex_color = unicode(desc_dict['Exterior Color'])
			if 'Interior Color' in desc_dict:
				c.int_color = unicode(desc_dict['Interior Color'])
			if 'Mode Code' in desc_dict:
				c.model_type = unicode(desc_dict['Model Code'])
			if 'Stock #' in desc_dict:
				c.stock_num = unicode(desc_dict['Stock #'])
			if 'Limit' in c.model:
				c.trim = 'Limited'
			else:
				c.trim = 'Premium'
			c.dealer = Dealer.gql('WHERE url = :1 ', url).get().key()
			self.store_car(c)
			return c.key()
		except AttributeError, e:
			logging.error('Error Parsing Car line %s \ne = %s', car_str, e) 
		return
```
<div>
<h4>Code Layout</h4>
	<div style="margin-left: 40px;"><ol>
		<li>File Structure</li>
		<ol>
			<li>Main.py - Main Request for loading jinja pages</li>
			<li>DataProcesses.py - basic store and processing data request classes</li>
			<li>Entities.py - Define all schema changes in one place for easy reference</li>
			<li>layout.html - main parent template for all jinja templates</li>
		</ol>
		<li> Libraries Used</li>
		<ol>
			<li><a href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup</a></li>
			<li><a href="http://twitter.github.io/bootstrap/">twitter-bootstrap</a></li>
			<li><a href="http://www.datatables.net/">data tables</a></li>
			<li><a href="https://developers.google.com/appengine/docs/python/overview">App Engine</a></li>
		</ol>
	</ol>
	</div>
</div>
<h2>The End...</h2>
<a href="/images/brz-scraps.png">{% img center /images/brz-scraps.png 650 415 BRZ Scraper %}</a>
If you skipped to the end, quick note I made a program that looked up all dealerships inventory and scraped their inventory so I can look up dealers with my prefferred model and trim at atleast basic price, without markups.

After running the program for a week I started calling dealerships and even with all the code in the world, it wasn't enough. Some dealers would advertise MSRP online and then when you called or came in there would be a $5000 dollar markup sticker next to the car. So I let the program sit. I finally made a few plans to go down to Southern California to pick up my car in a few weeks once some dealers got their shipments in from Subaru.

Monday May 13th, after giving up for another 2 weeks, I was looking over my app engine logs and I noticed my script would hit a bug on one of the latest dealers I had added to the list, I fixed the bug re-deploy and boom their it was the car I was looking for sitting in a local dealer nearby. I called the dealer immediately and within the next few hours I was signing my life away.

<h3>Last Day: Presentable tweaks</h3>

After acquiring my car I knew I wanted to tell the world about my story so I added more user admin features to limit people who could access and edit parts of the site in case others want to add more dealers to the scraping script.

Github Page https://github.com/laguayo/brz-scraps

App Engine Link: http://brz-scraps.appspot.com
