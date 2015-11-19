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
