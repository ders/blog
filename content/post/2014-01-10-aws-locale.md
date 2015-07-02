+++
date = "2014-01-10T14:55:00+09:00"
title = "AWS Locale"

+++

Every time I start a new EC2 Ubuntu instance, I'm confronted with the following
warning when I ssh in:

{{< highlight text >}}
_____________________________________________________________________
WARNING! Your environment specifies an invalid locale.
 This can affect your user experience significantly, including the
 ability to manage packages. You may install the locales by running:

   sudo apt-get install language-pack-UTF-8
     or
   sudo locale-gen UTF-8

To see all available language packs, run:
   apt-cache search "^language-pack-[a-z][a-z]$"
To disable this message for all users, run:
   sudo touch /var/lib/cloud/instance/locale-check.skip
_____________________________________________________________________
{{< /highlight >}}

Furthermore, a variety of package installations fail with some complaint
related to the locale, the default language, or both.
And for some reason the advice to install relevant language packs
is not helpful.

It turns out that there are some of environment variables
(`LANGUAGE`, `LC_CTYPE` and `LC_ALL` to be specific) that are not
set properly.

The advice to install language packs assumes that these environment
variables are set to a language that's not installed.  However, in the
case of a new EC2 instance, these variables are not set at all.

An easy way to get the warnings to go away is to edit the file
`/etc/default/locale` so that these variables always get set. I've found
that the default installation only sets `LANG`.

/etc/default/locale
{{< highlight text >}}
LANG=en_US.UTF-8
LANGUAGE=en_US
LC_CTYPE=en_US.UTF-8
LC_ALL=en_US.UTF-8
{{< /highlight >}}

As always, it's also a good idea to make sure you have the latest and
greatest packages:

{{< highlight text >}}
$ sudo apt-get update
$ sudo apt-get upgrade
{{< /highlight >}}

And finally, while we're at it, why not set the timezone?

{{< highlight text >}}
$ sudo dpkg-reconfigure tzdata
{{< /highlight >}}

Next time I need to set up a new EC2 instance,
I'll come read my own blog and know exactly what to do.

