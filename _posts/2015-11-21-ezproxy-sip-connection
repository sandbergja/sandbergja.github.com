---
layout: post
title: Using Evergreen ILS to authenticate EZProxy users using SIP2 and an SSH tunnel
tags: Evergreen, EZProxy
---

EZProxy now checks with Evergreen to see if a user can access our
databases from off campus.  This means that to log in to databases,
students and staff can use their X number (or OSU 9 number) and year
of birth.

Check it out here: http://ezproxy.libweb.linnbenton.edu:2048/login

This also means that anybody can troubleshoot off-campus database
access issues now by verifying DoBs and barcodes in Evergreen.  If a
patron has expired in Evergreen, they will no longer be able to access
databases from off-campus.  However, fines and overdues will not
impact a patron's ability to access databases.

I am going to update our procedures accordingly.  Also, just between
us, the old logins will continue to work until I de-activate them
(probably end of Fall term, when most everybody should be used to the
new way of logging in).


We have had some issues with people trying to login to EzProxy with a lower-case "x" in their X number.  Since EzProxy credentials and Evergreen credentials are now the same, patrons need to enter a capital X.

However, I just wrote a script that will force capitals when folks type their X number into the proxy server login.

Setting up SIP accounts on evergreen (oils_sip.xml, etc.)

I'd like to have our EZProxy server authenticate users using SIP2,
which is totally supported and documented here:
http://www.oclc.org/support/services/ezproxy/documentation/usr/sip.en.html.

However, I am not enthusiastic about sending unencrypted patron login
information over Telnet or raw sockets, and neither is our ILS
sysadmin.  I'd like to figure out a way to perform the SIP2
authentication/authorization check over SSH, but am not quite sure how
best to do that.  Do either of these approaches make sense?


Empowered SHD staff to troubleshoot EZProxy problems: 
https://docs.google.com/document/d/1doFapr1f2LoDNO11HQz_bADzBOn9DiP1pLqW3EFcEdA/edit#bookmark=id.mgzw9trk9hmb
