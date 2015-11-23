---
layout: post
title: Finding Unique Holdings using OCLC's Worldcat Search API
tags: OCLC python Evergreen
categories: systems
---


Created a report:
Displayed fields
* column renamed oclc: Bibliographic Record::Flattened MARC Fields::Normalized Value
* column renamed call_number: Bibliographic Record::Call Numbers::Call Number Label
* column renamed title: Bibliographic Record::Simple Record Extracts::Title Proper
Base Filters
* Bibliographic Record::Is Deleted = false
* Bibliographic Record::Call Numbers::Item::Circ lib is ours
* Bibliographic Record::Call Numbers::Item::Is deleted = false





`import csv, sys`
`import urllib.request as ur`
`from xml.etree import ElementTree as ET`

`if sys.argv[1]:
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
           csv_writer.writerow([row['oclc'], row['call_number'], row['title']])  `
