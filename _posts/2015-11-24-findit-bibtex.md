---
layout: post
title: Using BibTeX and citeproc to provide accurate citations in a Blacklight-based discovery layer
tags: Blacklight BibTeX
categories: Systems
---
##Background
A number of goals related to proper citations in WR courses:
- WR121 course objective: Cite sources using Modern Language Association (MLA) conventions, including a 'Works Cited' page.
- WR123 course objective: To help students use a systematic research process to present information in a format commonly assigned in college work.
- WR227 outcome: Collect and document information with footnotes and bibliographies.
- WR227 outcome: Use in-text citations and prepare bibliographies following the reseearch conventions in a particular discipline (for example, MLA, CBE, APA).

The Library also has a specific information literacy program outcome: use appropriate/current tools and technologies to create, produce and communicate.

As such, our information instruction sessions devote time to demonstrating use of tools that help to produce appropriate citations and teaching their limitations.

Blacklight includes a built-in citation feature for APA, MLA, and Chicago, based on some hard work from Stanford.  However, there are a number of mistakes in the built-in citations, particularly for non-book items.

###Citeproc

##Overview of a Citeproc-based workflow
Wanted most of this work done during index time, rather than at search time.
However, didn't want to rely on storing html or XML in MARC.  Also wanted to be able to switch citation styles according to demand,
taking advantage of CSL+citeproc flexibility.

1. Python script extracts relevant MARC data and creates BibTeX string.
2. BibTeX string is indexed in its own solr field.
3. Ruby script seeks out this solr field.  If it can't find one that it can parse, it generates one on the fly.
4. Ruby script converts the BibTeX string to CSL.
5. The citeproc gem eats CSL, and can format it into a citation in dozens of different formats.  It works its magic.
6. The formatted citation is displayed to the user.

