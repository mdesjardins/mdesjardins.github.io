---
title: L10N with Python in Avant Window Navigator Applets
author: mdesjardins
layout: post
permalink: /2007/10/30/l10n-with-python-in-avant-window/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2007/10/l10n-with-python-in-avant-window.html
categories:
  - blog
tags:
  - awn
  - i18n
  - l10n
  - python
---
<img src="/assets/images/file-788161.png" alt="" border="0" />It&#8217;s been a while since I blogged about my latest &#8220;hobby&#8221; (oh, how my wife would cringe), writing Python-based applets for the [Avant Window Navigator][1]. I've been spending a lot of time working on my [clock/calendar applet][2] lately, but today I'm 
going to go back to my [weather applet][3] because it's more interesting to write about.

A while ago, I decided it was time to add some alternate (that is, non-English) language 
support to the applet. The de-facto standard tool to accomplish this is the Python variant 
of GNU's [gettext][4]. I found a couple of resources that helped guide me through this. 
The first was the [wiki][5] for the "one laptop per child" project, which does a lot of 
L10N in Python. The second was an excellent [post][6] in the "Learning Python" blog (the author 
doesn't give his/her name in the [Who am I][7] section, otherwise I'd give props). And 
of course, there's always the official Python [library documentation][8], too.

The first step is to import the gettext libraries into your Python code, and do some setup work, thusly:

``` python
APP=">"awn-weather-applet"
DIR=os.path.dirname (__file__) + '/locale'
import locale
import gettext
locale.setlocale(locale.LC_ALL, '')
gettext.bindtextdomain(APP, DIR)
gettext.textdomain(APP)
_ = gettext.gettext
```

(Note: Made an edit to the above source code on Nov 16 2007 &#8211; evidently, the 
DIR parameter needs the full path to work properly)

Now, here&#8217;s what we&#8217;re actually doing. The call to <span style="font-family:courier new;">bindtextdomain</span> is <span style="font-style: italic;">binding</span> the <span style="font-weight: bold;">awn-weather-applet</span> domain to the <span style="font-weight: bold;">locale</span> subdirectory. When the applet is deployed, it has a locale directory right underneath the main script, weather.py. The directory argument of <span style="font-family:courier new;">bindtextdomain</span> is relative, so we&#8217;re pointing gettext at that directory. We&#8217;re basically telling gettext to look in the locale subdirectory for files named <span class="file"><var>language</var>/LC_MESSAGES/<var></var>awn-weather-applet.mo, where language is the two-letter language code defined by the environment (more on that later).

It should be noted that, customarily, locale files are stored in a default location which is a more global place in the filesystem, e.g. /usr/share/locale/<span style="font-style: italic;">language</span>/LC_MESSAGES. If you don&#8217;t supply a directory to the <span style="font-family:courier new;">bindtextdomain</span> method call, gettext will use the default directory for the system. I opted <span style="font-style: italic;">not</span> to use the default filesystem location because I wanted non-superusers to be able to easily use language files that I supply with the applet, using the standard AWN applet installation mechanism. Requiring the user to put .mo files in the /usr/share directory tree isn&#8217;t an option.


The call to <span style="font-family:courier new;">textdomain</span> sets the global domain to awn-weather-applet. Essentially we&#8217;re telling gettext that all future calls into gettext should use the </span><span class="file">awn-weather-applet domain as its source for translations.

Lastly, the line that reads <span style="font-family:courier new;">_ = gettext.gettext()</span> defines a convenient alias that we will use to identify strings in our code that need translation. If you&#8217;re familiar with i18n of C applications, this will look quite familiar; in the C implementation, an identically named macro is used for the same purpose.

The next step is to read through the code to identify the literal strings that will be translated. Generally, any UI element that a user will see is a candidate for translation. Things like log and console output are not as important. When we find a candidate string, we wrap it in _( ). For example, this:

``` python
self.dialog.set_title("Forecast")
```

becomes

``` python  
self.dialog.set_title(_("Forecast"))
```

Because of the alias at the top of the source file, what we&#8217;re <span style="font-style: italic;">really</span> doing is this:
  
``` python
self.dialog.set_title(gettext.gettext("Forecast"))
```
  
It&#8217;s a good thing we have the shorthand version!

As I alluded to earlier, gettext does its magic by reading files that ends in an .mo extension. At runtime, it reads 
the string that&#8217;s passed into it (in the above example, &#8220;Forecast&#8221;), then looks up that string in 
the proper .mo file to see if a translation is available. If it finds an entry, it returns the translated string, 
otherwise it just returns the string that was passed into it. So our next task is to create the .mo files.

Creating .mo files is a three step process. First, we need to run a tool that extracts all of the strings to be translated into a usable text file that a translator can edit. The tool to do this is called xgettext. The command line to run looks like this:

    xgettext --language=Python --keyword=_ --output=awn-weather-applet.pot *.py</pre>
  
This command generates an output file named awn-weather-applet.pot which acts as a template for translators to do their translation. The file has a bunch of lines that look like this:
  
```
#: weather.py:127
msgid "Forecast"
msgstr ""
```
  
So, let&#8217;s say we want to make a Spanish translation of the weather applet. First, we&#8217;d make 
a copy of this file, and name it something sensible like awn-weather-applet.po (note, you can also use 
the msginit tool, which does a few other housekeeping things for you like fill in the e-mail address). 
We&#8217;d find all the lines that start with msgid, translate it, and put the result in the following 
line&#8217;s msgstr, like this:

```
#: weather.py:127
msgid "Forecast"
msgstr "Pronóstico"
```
  
Next, we need to &#8220;compile&#8221; the awn-weather-applet.po file into an awn-weather-applet.mo file, so that it&#8217;s usable by gettext at runtime. This is done using the msgfmt command, like this:
  
    msgfmt awn-weather-applet.po -o awn-weather-applet.mo
  
That&#8217;s it! Now all you need to do is ensure that the awn-weather-applet.mo file ends up in the (base of weather applet)/locale/es/LC_MESSAGES directory, and Spanish translations work!

 [1]: http://wiki.awn-project.org/
 [2]: http://wiki.awn-project.org/index.php?title=Clock/Calendar_Applet
 [3]: http://wiki.awn-project.org/index.php?title=Weather_Applet
 [4]: http://docs.python.org/lib/module-gettext.html
 [5]: http://wiki.laptop.org/go/Python_i18n
 [6]: http://http//www.learningpython.com/2006/12/03/translating-your-pythonpygtk-application/
 [7]: http://www.learningpython.com/who-am-i/
 [8]: http://docs.python.org/lib/node732.html