---
layout: post
title: Getting XML holdings data out of the Evergreen ILS
tags: Evergreen
---

Here are some ways to get XML data out of the Evergreen ILS:

* supercat

The atom-full version of Supercat includes holdings data.

http://libcat.linnbenton.edu/opac/extras/supercat/retrieve/atom-full/record/{ID}

XPATH: //open-ils:volume[@opac_visible="t" and @deleted="f"][@lib="LBCCLIB"]//open-ils:status

where open-ils namespace is set to http://open-ils.org/spec/holdings/v1

* SRU

Record schema: MARCXML version 1.1
URL: http://libcat.linnbenton.edu/opac/extras/sru
Example Query URL (using Hemingway as the search term): http://libcat.linnbenton.edu/opac/extras/sru?version=1.1&operation=searchRetrieve&query=hemingway&maximumRecords=0
Some documentation about using SRU with Evergreen: http://evergreen-ils.org/dokuwiki/doku.php?id=evergreen-admin:sru_and_z39.50
Holdings data can be found in the 999


