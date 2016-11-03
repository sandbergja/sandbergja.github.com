---
layout: post
title: Getting XML holdings data out of the Evergreen ILS
tags: Evergreen
categories: Systems
---
As I work on discovery layer-related projects, I keep revisiting the question of what the best way to get holdings data from Evergreen is.  EG gives us a number of options here; here's what I've come across so far.

XML
===

There are several options in the XML realm:

Supercat
--------

The atom-full version of Supercat includes holdings data.

I get my XML from a URL like this: http://libcat.linnbenton.edu/opac/extras/supercat/retrieve/atom-full/record/257595

Where libcat.linnbenton.edu is the name of my host, and 257595 is the TCN of the record I'm interested in.

I then use a bit of XPATH -- something along these lines -- to actually grab the holdings data:

XPATH: //open-ils:volume[@opac_visible="t" and @deleted="f"][@lib="LBCCLIB"]//open-ils:status

Where open-ils namespace is set to http://open-ils.org/spec/holdings/v1 and LBCCLIB is the shortname for the branch I care about.  Note that Supercat returns a _lot_ of data, so it could be much faster.

OpenSearch
----------

A more documented standard!  Gives you pretty much the same data, though


http://libcat.linnbenton.edu/opac/extras/opensearch/1.1/-/marcxml-full/tcn/22222

UnAPI
-----

https://libcat.linnbenton.edu/opac/extras/unapi?id=tag:U2@bre:biblio_record_entry/280157&format=holdings_xml-full


JSON
----

I had a request that our Find It interface include due dates for holdings that are checked out.  Jason at Equinox suggested that I take a look at using OpenSRF to get these data:

https://libcat.linnbenton.edu/osrf-gateway-v1?service=open-ils.cat&method=open-ils.cat.asset.copy_tree.retrieve&param=[FAKE_SESSION_KEY]&param=294385&param=7

Where 294385 is a TCN and 7 is our ou id.

The session key can actually be anything you want; it isn't actually checked for authorization.  This is the approach I use for the [Evergreen Holdings Gem](https://github.com/sandbergja/evergreen_holdings_gem). Bill Erickson, Jason Stephenson, and Mike Rylander were kind enough to share some [really helpful details](http://georgialibraries.markmail.org/search/?q=%22Which+OpenSRF+gateway+should+I+use%3F%22#query:%22Which%20OpenSRF%20gateway%20should%20I%20use%3F%22%20list%3Aorg.georgialibraries.list.open-ils-dev%20order%3Adate-forward+page:1+state:facets) about the many opensrf gateways that you can use with Evergreen.
