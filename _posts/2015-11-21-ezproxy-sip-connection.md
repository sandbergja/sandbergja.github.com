---
layout: post
title: Using Evergreen ILS to authenticate EZProxy users using SIP2 and an SSH tunnel
tags: Evergreen EZProxy
categories: systems
---

## Summary

This post documents how Linn-Benton Community College set up an encrypted SIP2 connection between EZProxy and our Evergreen ILS.  This simplified library workflows and the number of passwords patrons needed to remember.

Now, to log in to databases from off-campus,
students and staff can use their X number (or OSU 9 number) and year
of birth, which is the same as Evergreen credentials.

## SIP setup

Our ILS sysadmin set up a SIP user for us to use.  This process is a bit complex, but is well documented in the [official Evergreen documentation](http://docs.evergreen-ils.org/2.7/_sip_server.html#_adding_sip_users).

how to test SIP

## EZProxy setup

I'd like to have our EZProxy server authenticate users using SIP2,
which is totally supported and documented here:
http://www.oclc.org/support/services/ezproxy/documentation/usr/sip.en.html.

Using log file to find the SIP requests that are going out.

However, I am not enthusiastic about sending unencrypted patron login
information over Telnet or raw sockets, and neither is our ILS
sysadmin.  I'd like to figure out a way to perform the SIP2
authentication/authorization check over SSH, but am not quite sure how
best to do that.  Do either of these approaches make sense?


{% highlight bash %}
#!/bin/bash
a=`/bin/ps -ef | /bin/grep ssh | /bin/grep -v grep | /bin/wc -l`
if [ $a -eq 0 ]; then
  echo "Restarting ssh tunnel"
  /usr/bin/ssh -L 6001:localhost:6001 sip_user@evergreen_server -f -N
fi
{% endhighlight %}
.ssh authorize without password


## Implications

This means that we no longer have to manually add EZProxy Users, which was a time-consuming workflow.

This also means that all reference and SHD staff can troubleshoot off-campus database
access issues now by verifying DoBs and barcodes in Evergreen.  If a
patron has expired in Evergreen, they will no longer be able to access
databases from off-campus.  However, fines and overdues will not
impact a patron's ability to access databases.

We have had some issues with people trying to login to EzProxy with a lower-case "x" in their X number.  Since EzProxy credentials and Evergreen credentials are now the same, patrons need to enter a capital X.

However, I just wrote a script that will force capitals when folks type their X number into the proxy server login.


Empowered SHD staff to troubleshoot EZProxy problems: 
https://docs.google.com/document/d/1doFapr1f2LoDNO11HQz_bADzBOn9DiP1pLqW3EFcEdA/edit#bookmark=id.mgzw9trk9hmb
