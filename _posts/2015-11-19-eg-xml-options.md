---
layout: post
title: Getting XML holdings data out of the Evergreen ILS
tags: Evergreen
---
As I work on discovery layer-related projects, I keep revisiting the question of what the best way to get holdings data from Evergreen is.  EG gives us a number of options here, so here are the options I've come across so far.

#XML

There are several options in the XML realm:

##Supercat

The atom-full version of Supercat includes holdings data.

I get my XML from a URL like this: http://libcat.linnbenton.edu/opac/extras/supercat/retrieve/atom-full/record/257595

Where libcat.linnbenton.edu is the name of my host, and 257595 is the TCN of the record I'm interested in.

I then use a bit of XPATH -- something along these lines -- to actually grab the holdings data:

XPATH: //open-ils:volume[@opac_visible="t" and @deleted="f"][@lib="LBCCLIB"]//open-ils:status

Where open-ils namespace is set to http://open-ils.org/spec/holdings/v1 and LBCCLIB is the shortname for the branch I care about.  Note that Supercat returns a _lot_ of data, so it could be much faster.

##SRU

Record schema: MARCXML version 1.1

URL: http://libcat.linnbenton.edu/opac/extras/sru

Example Query URL (using Hemingway as the search term): http://libcat.linnbenton.edu/opac/extras/sru?version=1.1&operation=searchRetrieve&query=hemingway&maximumRecords=0

http://libcat.linnbenton.edu/opac/extras/sru?version=1.1&operation=searchRetrieve&query=257595&maximumRecords=1

Some documentation about using SRU with Evergreen: http://evergreen-ils.org/dokuwiki/doku.php?id=evergreen-admin:sru_and_z39.50

Holdings data can be found in the 999

