---
layout: post
title: Finding Unique Holdings using OCLC's Worldcat Search API
tags: OCLC python Evergreen collections data-driven_decisions reports
categories: systems
---

While preparing for a major weeding project, our staff expressed a concern that we not weed any of our "unique" items.  These items, which may be held by only a handful of other libraries, are important to our Interlibrary Loan partners; we didn't want to weed anything that other libraries were counting on.

OCLC doesn't provide reports of unique items, so I created my own workflow for finding these data.  I ran a report in Evergreen to pull out titles, call numbers, and OCLC numbers:  

Displayed fields
* column renamed oclc: Bibliographic Record::Flattened MARC Fields::Normalized Value
* column renamed call_number: Bibliographic Record::Call Numbers::Call Number Label
* column renamed title: Bibliographic Record::Simple Record Extracts::Title Proper
Base Filters
* Bibliographic Record::Is Deleted = false
* Bibliographic Record::Call Numbers::Item::Circ lib is ours
* Bibliographic Record::Call Numbers::Item::Is deleted = false


I manually normalized the OCLC numbers (which appeared in a number of formats) using regular expressions in Notepad++.  I also took this opportunity to remove any non-OCLC control numbers that were lurking in our 035 fields.

I saved the results as a CSV folder and put it in the same directory as this little Python script.  It basically checks the CSV report against OCLC's Library Locations API.  It is a very crude script; it took over 10 hours to run and probably wasn't very kind to OCLC's servers.  However, it got us the data we needed.


<code>
import csv, sys
import urllib.request as ur
from xml.etree import ElementTree as ET

if sys.argv[1]:
   file_name = sys.argv[1]
else:
   file_name = 'report.csv'
   
export_file_name = 'unique.csv'

with open(file_name) as csvfile:
   with open(export_file_name, 'wt') as export_file:
      reader = csv.DictReader(csvfile)
      csv_writer = csv.writer(export_file, delimiter=',',quotechar='"', quoting=csv.QUOTE_MINIMAL)
      for row in reader:
         i = 0
         request_url = 'http://www.worldcat.org/webservices/catalog/content/libraries/' + row['oclc'].strip() + '?location=OUR_ZIP_CODE&wskey=LONG_STRING'
         root = ET.fromstring(ur.urlopen(request_url).read())
         holdings = root.findall('holding')
         for holding in holdings:
            i += 1
         if i < 3:
           csv_writer.writerow([row['oclc'], row['call_number'], row['title']])
</code>

This returned a list of 104 titles that had 0-2 other holdings in WorldCat.  Only 13 of these titles were in the section we were weeding, and none of them were titles that would meet our weeding criteria.  Looking at these data made us feel much safer in our weeding project, and gave us the confidence that we were not substantially limiting access to these resorces for patrons at our partner libraries. 
