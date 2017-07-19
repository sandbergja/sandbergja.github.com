---
layout: post
title: How we do Authority Control in Evergreen ILS
tags: Evergreen AuthorityControl
categories: Systems
---

We have been doing authority control in our Evergreen System since April 2016.  Evergreen's authority control module
has some interesting features, but it also is relatively new to Evergreen and could use [some more development](https://bugs.launchpad.net/evergreen/+bugs?field.tag=authority).
There are a few basic things (e.g. get a list of unauthorized headings in my bib records) that are not available
in the client interface, and that took me a bit of SQL experimentation to work.  I'm sharing our process here in the
hopes that it saves some time for someone else, and that it gets more eyes on our process to identify things we could
do better.

Starting out
============

We were fortunate to have some grant funding to pay [Backstage Library Works](http://bslw.com/) to do our basefile and
some current cataloging work.  This [2016 presentation](https://evergreen-ils.org/wp-content/uploads/2015/11/eg16-CatalogingForester_reduce.pptx)
by Galen Charlton, Mary Jinglewski, and Chad Cluff served as a guide through this process.

I was overly ambitious with how many bib records I could upload in a single queue, and I caused [pretty bad performance issues](https://bugs.launchpad.net/evergreen/+bug/1581552)
for my consortium (sorry, y'all).  After that, I used [MarcEdit](http://marcedit.reeset.net/)'s MarcSplit tool to split
up my files into files of 2,000 records or so.  Keeping each of those files in its own queue meant that I didn't cause any
more of those awful performance problems when I wanted to look at how a queue uploaded.

Backstage has given us a number of helpful reports, which we are still working through.  Several people from throughout
our library have been kind enough to help work through the issues that Backstage identified in those reports.

Backstage was also kind enough to put URIs into subfield $0 of authorized fields.  This was a priority for us, as it was
a relatively small step that helps us to prepare for a linked data environment.  Evergreen uses $0 for its own internal
linking purposes, so we swapped those URIs over to subfield $9 for now, but all those data are still there. :-D

Finding unauthorized headings
=============================

I have several SQL queries that help me identify headings in our bibliographic records that don't match any of our authority
records.  Most of them are pretty slow, and could probably be improved, but are better than nothing!

To catch unauthorized author headings:

    SELECT DISTINCT ent.value FROM metabib.browse_entry ent, metabib.browse_entry_def_map map,
    config.metabib_field f, biblio.record_entry b
    WHERE ent.id=map.entry
    AND map.def=f.id AND map.source=b.id AND NOT b.deleted
    AND f.field_class = 'author'
    AND ent.id NOT IN
    (SELECT DISTINCT entry FROM metabib.browse_entry_def_map WHERE authority IS NOT NULL);

This query actually works really really well!  Note that the field_classes are customizable, so you may have to run
`SELECT * FROM config.metabib_class` to see what you have available to you.

I use an almost identical query to catch unauthorized series title headings (I just remove all the headings with volume
numbers at the end):

    SELECT DISTINCT ent.value FROM metabib.browse_entry ent, metabib.browse_entry_def_map map,
    config.metabib_field f, biblio.record_entry b
    WHERE ent.id=map.entry
    AND map.def=f.id AND map.source=b.id AND NOT b.deleted
    AND f.field_class = 'series'
    AND ent.id NOT IN
    (SELECT DISTINCT entry FROM metabib.browse_entry_def_map WHERE authority IS NOT NULL)
    AND ent.value !~ '.*\d$';

Finally, to check 650$a subfields:

    SELECT DISTINCT trim(both from regexp_replace(value, '[[:punct:]]', ''))
    FROM metabib.full_rec m, biblio.record_entry b
    WHERE m.record=b.id AND NOT b.deleted AND tag ='650' AND subfield = 'a' AND ind2='0'
    AND trim(both from regexp_replace(value, '[[:punct:]]', '')) NOT IN
        (SELECT DISTINCT trim(both from regexp_replace(sort_value, '[[:punct:]]', ''))
        FROM authority.simple_heading)
    AND value NOT LIKE '%fictitious character%';
    
I throw the results into a [script that fetches matching authority records from the Library of Congress](https://github.com/sandbergja/dlc_authority_fetcher)
, and then review anything my script couldn't find to see whether the bib record needs to be corrected, or we need to make
a local authority, etc.

Adding more thesauri
====================

We have records from sources other the Library of Congress.  Each of them is pretty exciting in its own way.

Medical Subject Headings
------------------------

When we need a MeSH record, we usually just go to LocatorPlus, download the file, and upload it.  Rather than going through
the whole Marc Batch Import/Export process for every record, I usually just download them to a folder on my computer. When
I reach critical mass (maybe 30 or so records), I run the following command in Linux:

    cat Pwebrecon* > mesh_headings_to_load.mrc

If I'm on a Windows box, I run the following in the command prompt (it doesn't work in Powershell):

    type Pwebrecon* > mesh_headings_to_load.mrc

I then use the following steps in Marc Batch Import/Export to load the file:

1. Set Record Type=Authority records
2. Give the queue a name
3. Set Record match set = LBCC AUTH
4. Set Record Source=National Library of Medicine
5. Import non-matching records should be checked, all others unchecked
6. Choose the authority file and upload

The LBCC AUTH record match set is very simple, just matching on the normalized heading.

Queens Library Spanish Headings
-------------------------------

The Queens Borough Public Library is fantastic!  You can download their Spanish translation of LCSH for no charge from
[lcsh-es.org](http://lcsh-es.org).  We uploaded almost all of this file into Evergreen; we just left out geographic
headings and fictitious characters, because they are likely to be the same as the English terms.

Thanks to everybody at the Queens Library and lcsh-es for providing this!

Local Records
-------------

We also make our own authority records!  At first, we were making a lot of records for validation purposes.  However,
this was time-consuming, and there are a few bugs that affect the order of bib subfields around in authorized fields
_anyway_, so there wasn't much benefit to creating records that specify those subfields and their order.

Updating LCSH
=============

LCSH change on a monthly basis, and we try to keep up.  Unfortunately, so many of the LC's changes lately have been
switching 150s to 100s, and Evergreen isn't able to help too much with this process (it does not attempt to change bib
tags, so it would never swap a 650 into a 600).  I'd still like to see a better workflow for these updates.

In general, for X50 to X00 changes, I go to the
[catalog browse interface](http://libcat.linnbenton.edu/eg/opac/browse?locg=8) to see how many records I would have
to change.  If it's only a handful, I just fix them immediately.  If it's more than that, I create a record bucket,
download those bib records, fix them in MarcEdit, and re-upload to Evergreen.

Using our authority data
========================

Our [discovery layer](http://libfind.linnbenton.edu/) uses authority data to help improve searches (especially when
a patron uses a synonym or alternate form of a name in their query).  The authority record ID in the bibliographic $0
makes this pretty easy!  As our indexing script goes through our MARC records, if it hits a $0 in an authorizable field,
it uses supercat to fetch the authority record, and uses various authority fields (mainly the 4XX) to enrich those
records.  Supercat makes this easy, with a URL in the following format: `http://libcat.linnbenton.edu/opac/extras/supercat/retrieve/marcxml/authority/AUTHORITY_RECORD_ID`

Next Steps
==========

We will always have cleanup of our authority file to do.  Yesterday, I noticed that we have a fair number of duplicate
authority records in which the only difference is the presence of a death date in one of the records.

I'd like to further refine the process we use when LCSH terminology is hostile to our patrons.  We've used our local
thesaurus in one major way so far (our catalog uses terms like _Undocumented immigrants_ and _Children of undocumented immigrants_
instead of the current LCSH terms), but there are plenty of other LCSH terms that still need some examination.

I'm also hoping to figure out some ways to assess this authority work so that we can spend our time doing the most
high-impact authority work we can do.

I'm also heartened to see interest in more development for Evergreen's authority module.  Evergreen does have a pretty
solid framework for authority control, and I look forward to seeing what the future holds. 
