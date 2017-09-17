# Web Scraping with Python: An Introductory Tutorial

#### Rob Osterburg 
##### Software Engineer / Instructor

Note: This tutorial introduction to web scaping with python 3 using requests and BeautifulSoup that was presented at [Denver Data Science Day 2017](http://denverdatascienceday.com/). 

## Acknowledgements

* The generous [sponsors](http://denverdatascienceday.com/index.php/sponsors/) of [Denver Data Science Day 2017](http://denverdatascienceday.com/)

* [Galvanize](https://www.galvanize.com/pick-a-location?page=%2F) for hosting [Denver Data Science Day 2017](http://denverdatascienceday.com/)

* Bob Mickus, Tyler B. and all the volunteers from [PyData Denver](https://www.meetup.com/PyData-Denver/) who organized [Denver Data Science Day](http://denverdatascienceday.com/)

* Miguel Grinberg whose [Easy Web Scraping with Python](https://blog.miguelgrinberg.com/post/easy-web-scraping-with-python) blog post inspired this tutorial.  Miguel's posted his tutorial in 2014 and PyVideo.org has recently undergone a significant revision.

## Topics
* Which Python packages to use
* Cover key concepts, tools and techniques
* Scrape some data 
    - Organize it using a namedtuple
    - Persist it as a JSON file
    - Read it back into a namedtuple and demonstrate semantic equivilence 
* Share recommended resources

## Key Packages

* requests -- Issues HTTP requests to web servers and handles the response 

* beautifulsoup4 -- Parse the returned web page and makes it searchable

* These packages are *not* in Python's standard library, here is how to install them:
 
    - Standard Python: `pip install requests beautifulsoup4`

    - Anaconda Python: `conda install requests beautifulsoup4`



```python
# Requests is 'HTTP for Humans'
import requests

# BeautifulSoup parses and builds tree from HTML
from bs4 import BeautifulSoup
```

## Warning! Warning!
* Only scrape as a last resort, first see if the site has an API or other means of accessing their data
* Web scraping is commonly frowned upon by the site's owners
* **Always check the site's Terms of service, Conditions of use, and robots.txt file**   
* Aggressive scrapers can take a site down, owner's can rightfully consider that a denial of service attack
* If in doubt, talk to a lawyer, especially for *anything* work related
* Watch [Sustainable Scrapers, PyData DC, 2016](http://pyvideo.org/pydata-dc-2016/sustainable-scrapers.html) to understand how a data journalist deals with these issues. It's a good talk and quite entertaining -- in a geeky way

## PyVideo.org -- Is it OK to scrape?
* [PyVideo.org](http://pyvideo.org) provides valuable meta-data about Python presentations
* It makes finding talks easy and can be searched by event, presenter, or topic 
* But what restrictions does it have against scraping?  
* Let's take a look

## PyVideo.org - Terms and Conditions?

None that I can see, the site's [About page](http://pyvideo.org/pages/about.html) says:

> ... PyVideo.org is a freely available index of freely available resources that seek to provide everyone with the opportunity to learn about Python.

## PyVideo.org - Robots.txt?
* A robots.txt files contain *restrictions* on the content that web crawlers are permitted to access 

* The [http://pyvideo.org/robots.txt](http://pyvideo.org/robots.txt) is empty, except for one comment 

* It appears that there are no restrictions on scraping PyVideo.org  

* Let's minimize our impact on the site 

* To see *The Robots Exclusion Protocol* visit [http://www.robotstxt.org/](http://www.robotstxt.org/).


```python
# Getting the Page
resp = requests.get('http://pyvideo.org/tags.html')

# A status code other than 200 indicates problem of some sort
print('status code:', resp.status_code)

# A better approach is let requests throw an exception when a problem occurs
resp.raise_for_status()
```

    status code: 200


## HTTP Responses

HTTP Verb | Effect | Success | Failure
--------- | ------ | ------- | --------
POST      | Create | 200     | 400, 40X, 500
**GET**       | **Read**   | **200**     | **400, 40X, 500**
PUT       | Update | 200     | 400, 40X, 500 
DELETE    | Delete | 200     | 400, 40X, 500

* [Comprehensive list of HTTP status codes](https://httpstatuses.com/)


```python
# Parse the Page
soup = BeautifulSoup(resp.text, 'html.parser')
```

 ## BeautifulSoup - Parses the page
 
* Models the page as a tree
* Similar to the Document Object Model
 ![tags page structure](./images/htmltree.png)

* Parsers build the tree, pick one
 - html.parser -> Default parser for Python 3
 - HTMLParser  -> Default parser for Python 2
 - lxml        -> Fast requires installing a C library and a PyPI package 
 - html5lib    -> Pure Python and part of the standard library
 
 cite: https://interactivepython.org/runestone/static/pythonds/Trees/ExamplesofTrees.html

## BeautifulSoup - Well documented and easy to use API
* [select](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#css-selectors)('css selector') --> [List of Tags]

* [find_all](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all)(tags, keyword_args, attrs={'attr', 'value'})  --> [List of Tags]
    
* [find](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find)(tags, keyword_args, attrs={'attr', 'value'}) --> Tag

* [BeautifulSoup 4 Documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

## Understanding Page Structure

* To extract data from a page, you need to understand its structure
* BeautifulSoup holds the tree in memory
* Look into it using the our browser's built-in developer tools
    - Mac -- Cmd + Opt + i
    - Linux -- Ctrl + Shift + i
    - Windows -- Ctrl + Shift + i

Visiting the [Tags page](http://pyvideo.org/tags) you can see the structure of the page  
![tags page structure](./images/structure-of-tags-page.png)

## Developer Tools - Getting a CSS selector
* Use the element inspector to locate items you want to scrape
* Right-click on the element > Copy > Copy CSS Selector
* Paste the selector into your code
* Make it more general by removing aspects that specific to *a* particular tag
* In this case, `nth-child()` and `.col-md-3`
* The idea here is to tune the selector to the data you want to scrape
* Tutorial: [An Intro to CSS: Finding CSS Selectors](https://dailypost.wordpress.com/2013/07/25/css-selectors/)


```python
# Build a list of anchor elements using a CSS selector
# First we need to clean our CSS using a for-loop
topics = []
as_copied_css = 'li.col-md-3:nth-child(1174) > a:nth-child(1)'
cleaned_css = 'li > a'
for topic in soup.select(cleaned_css):
    topics.append(topic)
topics[:8]
```




    [<a href="http://pyvideo.org/index.html"><i class="fa fa-fw fa-home"></i> <span>Start</span></a>,
     <a href="http://pyvideo.org/events.html"><i class="fa fa-fw fa-list-ul"></i> <span>Events</span></a>,
     <a href="http://pyvideo.org/tags.html"><i class="fa fa-fw fa-tags"></i> <span>Tags</span></a>,
     <a href="http://pyvideo.org/speakers.html"><i class="fa fa-fw fa-users"></i> <span>Speakers</span></a>,
     <a href="http://pyvideo.org/pages/about.html"><i class="fa fa-fw fa-info"></i> <span>About</span></a>,
     <a href="http://pyvideo.org/pages/thank-you-contributors.html"><i class="fa fa-fw fa-info"></i> <span>Thank You</span></a>,
     <a href="http://pyvideo.org/pages/thanks-will-and-sheila.html"><i class="fa fa-fw fa-info"></i> <span></span></a>,
     <a href="http://pyvideo.org/tag/2to3/">2to3</a>]



## Why List Comprehensions?
* List comprehensions are a powerful way to create lists
* For-loop code above == List comprehension below
* The CSS selector got us 90% of the way there
* List comprehension can do the rest
* I find them very handy, I think you will too
* Tutorial: [Understanding List Comprehensions in Python 3](https://www.digitalocean.com/community/tutorials/understanding-list-comprehensions-in-python-3)


```python
# Now let's using a list comprehension 
# to build the list of links
topics = [topic                                        # adds the topic to the list
          for topic in soup.select('li > a')]          # loop over Tags from select
topics[:8]
```




    [<a href="http://pyvideo.org/index.html"><i class="fa fa-fw fa-home"></i> <span>Start</span></a>,
     <a href="http://pyvideo.org/events.html"><i class="fa fa-fw fa-list-ul"></i> <span>Events</span></a>,
     <a href="http://pyvideo.org/tags.html"><i class="fa fa-fw fa-tags"></i> <span>Tags</span></a>,
     <a href="http://pyvideo.org/speakers.html"><i class="fa fa-fw fa-users"></i> <span>Speakers</span></a>,
     <a href="http://pyvideo.org/pages/about.html"><i class="fa fa-fw fa-info"></i> <span>About</span></a>,
     <a href="http://pyvideo.org/pages/thank-you-contributors.html"><i class="fa fa-fw fa-info"></i> <span>Thank You</span></a>,
     <a href="http://pyvideo.org/pages/thanks-will-and-sheila.html"><i class="fa fa-fw fa-info"></i> <span></span></a>,
     <a href="http://pyvideo.org/tag/2to3/">2to3</a>]



* Now lets look at one of the elements in our list
* Each element is a BeautifulSoup Tag object
* *Tag* allows access to all the element's data including
    - *attributes* that are accessed using dictionary like syntax
    - *contents* that are displayed on the page


```python
# Let's look a Tag instance
topic = topics[7]
# Access the data it holds
type(topic), topic, topic.contents, topic['href']
```




    (bs4.element.Tag,
     <a href="http://pyvideo.org/tag/2to3/">2to3</a>,
     ['2to3'],
     'http://pyvideo.org/tag/2to3/')



* List comprehensions are concise and powerful to process data
* From the list of anchor elements
* Let's create a list of links to topics pages
* Filtering out all unrelated links


```python
# Extracting and filtering the links
topics = [topic['href']                       # extract attribute value
          for topic in soup.select('li > a')  # loop over anchors
          if '/tag/' in topic['href']]        # filter
         
len(topics), type(topics[0]), topics[:5]
```




    (1461,
     str,
     ['http://pyvideo.org/tag/2to3/',
      'http://pyvideo.org/tag/3d/',
      'http://pyvideo.org/tag/3d-printing/',
      'http://pyvideo.org/tag/abc/',
      'http://pyvideo.org/tag/accelerate/'])




```python
# Get the page
resp = requests.get('http://pyvideo.org/tag/scraping')
resp.status_code
```




    200




```python
# Parse the page
soup = BeautifulSoup(resp.text, 'html.parser')
```

## Using Tag.find_all()
* We have used `select()`, not let's try out `find_all()` 
* Our goal is create a list of the links to all the talks on a particular topic


```python
# Find all the video page links using find_all()
h4_tags = soup.find_all('h4', class_='entry-title')
h4_tags[0]
```




    <h4 class="entry-title">
    <a href="/pydata-dc-2016/open-data-dashboards-python-web-scraping.html" rel="bookmark" title="Permalink to Open Data Dashboards &amp; Python Web Scraping">
               Open Data Dashboards &amp; Python Web Scraping
            </a>
    </h4>



## BeautifulSoup in action
* Methods: `find_all()` and `select()` both return list of Tag objects
* Accessing descendents:  Tag.Descendent_Tag['attribute']


```python
# Extracting the link from <a> tag inside the <h4>
findall_talks = [tag.a['href'] for tag in h4_tags]
len(findall_talks), findall_talks[0]
```




    (13, '/pydata-dc-2016/open-data-dashboards-python-web-scraping.html')




```python
# Same thing using select() -- In this case select seems cleaner
site_url = 'http://pyvideo.org'
select_talks = [site_url + tag['href'] 
                for tag in soup.select('article > section > h4 > a')]
len(select_talks), select_talks[0]
```




    (13,
     'http://pyvideo.org/pydata-dc-2016/open-data-dashboards-python-web-scraping.html')



## Moving on to extract presentation meta data
* Our ultimate destination is now in sight -- The video's presentation page
* It's the talk page -- that's where the best data are found


```python
# Get a page and parse it 
resp = requests.get(select_talks[0])
resp.raise_for_status()
soup = BeautifulSoup(resp.text, 'html.parser')
```


```python
# Extract the talk's title
titles = [title.contents[0].strip() 
          for title in soup.select('.entry-title > a')]
title = titles[0]
title
```




    'Open Data Dashboards & Python Web Scraping'




```python
# Extract the speaker's name 
names = [elem.contents[0] 
         for elem in soup.select('.url')]
names
```




    ['Marie Whittaker']




```python
# Extract the talk details
details = [detail 
           for detail in soup.select('.details-content > ul > li > a')]
details
```




    [<a href="http://pyvideo.org/events/pydata-dc-2016.html">PyData DC 2016</a>,
     <a href="https://www.youtube.com/watch?v=kc676iLvib8" rel="external">YouTube</a>,
     <a href="http://pyvideo.org/tag/data/">Data</a>,
     <a href="http://pyvideo.org/tag/scraping/">scraping</a>,
     <a href="http://pyvideo.org/tag/web/">web</a>]




```python
# Extract the YouTube link
links = [detail['href'] 
         for detail in details 
         if 'youtube.com' in detail['href']]
link = links[0]
```


```python
# Extract the subject tags
tags = [link.contents[0].lower() 
        for link in details 
        if 'tag' in link['href']]
tags
```




    ['data', 'scraping', 'web']




```python
# Extract the description
paragraphs = soup.select('.entry-content > p')
description = '\n\n'.join([p.contents[0] for p in paragraphs])
print(description)
```

    PyData DC 2016
    
    Distilling a world of data down to a few key indicators can be an effective way of keeping an audience informed, and this concept is at the heart of a good dashboard. This talk will cover a few methods of scraping and reshaping open data for dashboard visualization, to automate the boring stuff so you have more time and energy to focus on the analysis and content.
    
    This talk will cover a basic scenario of curating open data into visualizations for an audience. The main goal is to automate data scraping/downloading and reshaping. I use python to automate data gathering, and Tableau and D3 as visualization tools -- but the process can be applied to numerous analytical/visualization suites.
    
    I'll discuss situations where a dashboard makes sense (and when one doesn't). I will make a case also that automation makes for a more seamless data gathering and updating process, but not always for smarter data analysis.
    
    Some python packages I'll cover for web scraping and downloading/reshaping open data include: openpyxl, pandas, xlsxwriter, and BeautifulSoup. I'll also touch on APIs.


##  Saving the Data

* Use namedtuple to capture information about PyVideo talk

* What do we need to capture
    - Talk title (string)
    - Name of presenter(s) (list)
    - Description (string)
    - Tags (list)
    - Link to video (string)
    
* Let's use JSON 
    - Awesome for persisting structured data 
    - Excellent data exchange format
    

## Why namedtuples?
* namedtuples are a lightweight container for data
* Using a name to access data elements is their advantage
* Numeric indexing is all that regular tuple provides


```python
# Storing related data in a namedtuple
from collections import namedtuple

# create the structure
pyvideo = namedtuple('pyvideo', 'title names description tags link')

# create an instance
talk_data = pyvideo(title=title, names=names, tags=tags, link=link, description=description)

# access the data and display it as a formatted string
fmt = 'title={}\nnames={}\ntags={}\nlink={}'
print(fmt.format(talk_data.title, talk_data.names, talk_data.tags, talk_data.link))
```

    title=Open Data Dashboards & Python Web Scraping
    names=['Marie Whittaker']
    tags=['data', 'scraping', 'web']
    link=https://www.youtube.com/watch?v=kc676iLvib8


## Why JSON?
* Handles structured data well
* Excellent data exchange format
* Python supports round-tripping between namedtuples and JSON
    - namedtuple objects can be saved a JSON formatted text
    - JSON formatted text can be convert back to namedtuple objects
    - Of course, text can be written to file 


```python
# Writing the namedtuple to disk as JSON
import json

# Write to file
with open('a_video.json', 'w') as fout:
    # Note: preserve keys by saving a dictionary, not a list
    json.dump(talk_data._asdict(), fout)
    
# Read from file
with open('a_video.json', 'r') as fin:
    restored_talk_data = json.load(fin)

# Deserialize -- Note: pass as keyword arguments using **
restored_talk_data = pyvideo(**restored_talk_data)

# Compare semantically
talk_data == restored_talk_data
type(talk_data), type(restored_talk_data)
```




    (__main__.pyvideo, __main__.pyvideo)



## Writing web scraping code has advantages

* Excellent source of valuable data 

* The code is approachable

## ... and presents some problems too

* Your code breaks when the site is updated

* Read the terms and conditions, otherwise ...

* Getting blocked, banned, sued ...

## Resources
1. [Automate the Boring Stuff](https://automatetheboringstuff.com/), [Chapter 11 — Web Scraping](https://automatetheboringstuff.com/chapter11/) by Al Sweigart (Free PDF version online).  Takes you through topics step-by-step, includes using Selenium to fill out forms and simulate mouse clicks. 

1. [RealPython Blog -- Web Scraping With Scrapy and MongoDB](https://realpython.com/blog/python/web-scraping-with-scrapy-and-mongodb/) by Micheal Herman. Scrapy is a Python package that makes scraping code easier to maintain. 

1. [Talk Python to Me Podcast -- Web scraping at scale with Scrapy and ScrapingHub](https://talkpython.fm/episodes/show/50/web-scraping-at-scale-with-scrapy-and-scrapinghub). Web scraping as a Service from the author of Scrapy.

1. [PyVideo.org](http://pyvideo.org/)— Comprehensive catalog of videos of over 8000 of Python related presentations. Talks on scraping web pages can be found on the [Scraping page](http://pyvideo.org/tag/scraping/). 

1. [Web Scraping with Python: Collecting Data from the Modern Web](https://www.amazon.com/Web-Scraping-Python-Collecting-Modern/dp/1491910291) by Ryan Mitchell.  This 4.5 star book on Amazon covers scraping topics in depth.

1. [Awesome Python](https://awesome-python.com/) -- PyPI has over 100,000 packages.  Awesome Python is a curated list of the best, see their recommended web scraping packages [here](https://awesome-python.com/#web-crawling).

1. Other Practice Sites
    * [Books to Scrape](http://books.toscrape.com/)
    * [Quotes to Scrape](http://quotes.toscrape.com/) 
