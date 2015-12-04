---
layout: post
title: Using BibTeX and citeproc to provide accurate citations in a Blacklight-based discovery layer
tags: Blacklight BibTeX
categories: Systems
---
##Background
LBCC's writing courses have a number of goals related to proper citations:

- WR121 has the following course objective: "Cite sources using Modern Language Association (MLA) conventions, including a 'Works Cited' page."
- WR123 has the following course objective: "To help students use a systematic research process to present information in a format commonly assigned in college work."
- WR227 has two relevant outcomes: "Collect and document information with footnotes and bibliographies," and "Use in-text citations and prepare bibliographies following the research conventions in a particular discipline (for example, MLA, CBE, APA)."

As such, our information instruction sessions devote time to demonstrating use of tools that help to produce appropriate citations and teaching their limitations.  Much of my work customizing our Blacklight interface centered around how my colleagues and I would teach students the proper use of the tool.  I began to check up on Blacklight's citation features to see how I would teach them.

Blacklight's built-in citation feature creates APA, MLA, and Chicago citation, thanks to some hard work from developers at Stanford.  However, these built-in citations contain countless formatting mistakes, particularly for non-book items.

###Citeproc
The Ruby implementation of Citeproc converts Citation Style Language (CSL) strings into properly formatted citations for dozens of different citation standards.  These technologies form the basis of popular citation software like Zotero and Mendeley.  I wanted to harness some of the [Citeproc gems](https://github.com/inukshuk/citeproc)' power and flexibility to improve the citations I was seeing. 

##Overview of a Citeproc-based workflow
To save users' time, I wanted to run as much of this process while documents are initially indexed into the collection, rather than when the user actually wants to create a citation.
However, I felt nervous about storing fully formatted HTML or XML in the MARC files I indexed.  I also wanted to retain the flexibility of switching citation styles according to demand without having to re-index our entire collection.  I decided to store information in the very simple (but descriptive) BibTeX format during index time, leaving Citeproc to run its magic when users requested a citation.

Here is the workflow I settled on:

1. [Python script](https://github.com/sandbergja/blacklight_ingest_scripts/blob/master/bibtex_functions.py) extracts relevant MARC data and creates BibTeX string.
2. BibTeX string is indexed in its own solr field.
3. [Ruby helper script](https://github.com/sandbergja/discovery_layer/blob/master/app/helpers/citation_helper.rb) seeks out this solr field.  If it can't find one that it can parse, it generates one on the fly.
4. Ruby script converts the BibTeX string to CSL.
5. The citeproc gem eats CSL and creates a properly formatted citation in the styles that we specify.
6. The formatted citation is displayed to the user.

