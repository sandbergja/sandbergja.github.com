---
layout: post
title: Using Evergreen ILS to authenticate EZProxy users using SIP2 and an SSH tunnel
tags: Evergreen EZProxy
categories: systems
---

## Summary

This post documents how Linn-Benton Community College set up an encrypted SIP2 connection between EZProxy and our Evergreen ILS.  This simplified library workflows and the reduced the number of passwords students need to remember.

Now, to log in to databases from off-campus, students and staff can use the same credentials they use to log in to their Evergreen accounts: their X number (or OSU 9 number) and year
of birth.

## SIP setup

Our ILS sysadmin set up a SIP user for us to use.  This process is a bit complex, but is well documented in the [official Evergreen documentation](http://docs.evergreen-ils.org/2.7/_sip_server.html#_adding_sip_users).

## EZProxy setup

I spent a lot of time testing the output to make sure that EZProxy was sending SIP messages correctly.  I used the Debug directive in my user.txt file to send all sent and received SIP2 communications in messages.txt. 

OCLC provides some basic EZProxy documentation.  These include [how to use user.txt to authenticate users using SIP](http://www.oclc.org/support/services/ezproxy/documentation/usr/sip.en.html).

However, I was not enthusiastic about sending unencrypted patron login information over Telnet or raw sockets, and neither was our ILS
sysadmin.  Therefore, I set up an SSH tunnel to encrypt all SIP2 communication between the two servers.

I first [set up an authentication key](http://www.debian-administration.org/article/530/SSH_with_authentication_key_instead_of_password) so that I didn't have to worry about any passwords for the SSH tunnel.

I then wrote a very brief Cygwin script that checks to see if the tunnel process is still running.  If the tunnel is not running, it opens it again.  I set this script to run every minute as a Windows scheduled task.  I originally used autossh instead of ssh, which sends occassional test packages back and forth to make sure that the tunnel is still operational.  However, the test packages were taking up too much bandwidth, so I stuck with plain old ssh.

{% highlight bash %}
#!/bin/bash
a=`/bin/ps -ef | /bin/grep ssh | /bin/grep -v grep | /bin/wc -l`
if [ $a -eq 0 ]; then
  echo "Restarting ssh tunnel"
  /usr/bin/ssh -L 6001:localhost:6001 sip_user@evergreen_server -f -N
fi
{% endhighlight %}


## Implications

This setup allowed us to skip the laborious process of manually adding EZProxy user information from a Banner report into user.txt.  This saves us a lot of time, particularly during the add/drop period at the beginning of term.

This also means that all staff at the Reference and Student Help Desks can troubleshoot off-campus database
access issues.  Rather than a complicated process that included referral to a faculty librarian, front-line staff can now simply [verify patrons' birth dates and barcodes in Evergreen](https://docs.google.com/document/d/1doFapr1f2LoDNO11HQz_bADzBOn9DiP1pLqW3EFcEdA/edit#bookmark=id.mgzw9trk9hmb).  If a
patron has expired in Evergreen, they will no longer be able to access
databases from off-campus.  However, fines and overdues will not
impact a patron's ability to access databases.

Since Evergreen -- unlike EZProxy's built-in authentication -- is case sensitive, we had several patrons try to log in to EzProxy with a lower-case "x" in their X number.  I added a very simple script to the user field in docs/login.htm that forces capitals when folks type their X number into the proxy server login:

{% highlight html %}
<input type="text" name="user" id="user"
  onkeypress="javascript:{this.value=this.value.toUpperCase();}" />
<label for="user"> Your X number (or OSU student ID number)</label>
{% endhighlight %}
