---
layout: post
title: Getting XML holdings data out of the Evergreen ILS
tags: Evergreen
categories: Systems
---
As I work on discovery layer-related projects, I keep revisiting the question of what the best way to get holdings data from Evergreen is.  EG gives us a number of options here; here's what I've come across so far.

##XML

There are several options in the XML realm:

###Supercat

The atom-full version of Supercat includes holdings data.

I get my XML from a URL like this: http://libcat.linnbenton.edu/opac/extras/supercat/retrieve/atom-full/record/257595

Where libcat.linnbenton.edu is the name of my host, and 257595 is the TCN of the record I'm interested in.

I then use a bit of XPATH -- something along these lines -- to actually grab the holdings data:

XPATH: //open-ils:volume[@opac_visible="t" and @deleted="f"][@lib="LBCCLIB"]//open-ils:status

Where open-ils namespace is set to http://open-ils.org/spec/holdings/v1 and LBCCLIB is the shortname for the branch I care about.  Note that Supercat returns a _lot_ of data, so it could be much faster.

###OpenSearch
A more documented standard!  Gives you pretty much the same data, though


http://libcat.linnbenton.edu/opac/extras/opensearch/1.1/-/marcxml-full/tcn/22222


##JSON

http://libcat.linnbenton.edu/gateway?service=open-ils.cat&method=open-ils.cat.asset.copy_tree.retrieve&param=[AUTH TOKEN]&param=294385&param=7

Where 294385 is a TCN and 7 is our ou id.

To get an auth token Within the staff client, press control+shift+f7 to get a command line interface at the top of the screen (as long as you have the DEBUG_CLIENT permission). Enter the command `ses()`, and you will get a token that you can use.

The gateway there is actually an older mechanism for external clients to access OpenSRF, and newer code uses osrf-http-translator, but that service expects you to POST your requests instead of GET them and the format is more complicated.
