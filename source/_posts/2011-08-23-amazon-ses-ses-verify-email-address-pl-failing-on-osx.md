---
title: Amazon SES, ses-verify-email-address.pl failing on OSX
author: admin
layout: post
permalink: /2011/08/23/amazon-ses-ses-verify-email-address-pl-failing-on-osx/
categories:
  - blog
---
I spent the better part of last night trying to get Amazon SES working from my Mac. After 
reading the README, installing all the dependencies, and doing everything correctly, I 
kept getting the following error:

    Can't locate object method "ssl_opts" via package "LWP::UserAgent" at SES.pm line 249.

The only Google hits were for people trying to set up SES with Ubuntu or Debian and weren't 
much help. It finally dawned on me what was going on this morning... when I did a "which perl", 
I noticed that it was running from my MacPorts directory:  

    ~/Downloads/amazon-ses> which perl
    /opt/local/bin/perl

However, when I looked at the code in the script, the "she-bang" at the top of the script 
was pointing to the perl in /usr/bin, which was installed with the operating system:  

    #!/usr/bin/perl -w

So all of my dependency installations were ending up in my MacPorts version of Perl, but 
the script was executing using the OSX version of Perl, and nothing worked. Changing the 
first line to  

    #!/opt/local/bin/perl -w

fixed the problem for me. It probably would make even more sense for me to make /usr/bin/perl 
a symbolic link to my MacPorts installation. I hope this can help someone else out there 
suffering through the same problem!