---
layout: post
title: Getting XML holdings data out of the Evergreen ILS
tags: Blacklight data-driven_decisions
categories: Systems
---
Using the [ahoy](https://github.com/ankane/ahoy) rails gem, I am able to collect very detailed analytics data about how [Find It](https://libfind.linnbenton.edu:4430) (our discovery layer) is used.

To inform information literacy instruction and publicity:
* A monthly count of unique visitors
* A monthly count of searches

To inform information literacy instruction:
* Average length (in minutes) for each session

To inform collection development work and search tuning:
* A quarterly list of searches that return 0-5 results
* A monthly report that includes % of total searches that return under 10 results

To inform interface design and information literacy instruction
* Relative popularity of certain facets
* % of searches that use facets

To determine whether or not additional qualitative assessment is required:
* Percentage of users who return within three months
