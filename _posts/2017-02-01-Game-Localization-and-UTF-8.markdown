---
layout: post
title:  "Game Localization and UTF-8"
date:   2017-02-01
categories: architecture strings serialization
---
After some research I have decided a pretty good way to localize your game is to store all text in text files, in [UTF-8 format](http://utf8everywhere.org/). UTF-8 is super widely used and 100% backwards compatible with ASCII. Also the clever design of the UTF-8 encoding lets programmers treat UTF-8 buffers as naive byte arrays. UTF-8 buffers can be naively sorted and still retain valid results, as an example!

![utf8_web_growth](/assets/utf8_web_growth.png)

However sometimes UTF-16 is needed to deal with Windows APIs… So some conversions from UTF-8 to UTF-16 can be pretty useful. To make things worse there doesn’t exist good information on encoding/decoding UTF-8 and UTF-16, other than the original RFC documents. So in swift fashion I hooked up a tiny header with the help of [Mitton’s UTF-8 encoder/decoder from tigr](https://bitbucket.org/rmitton/tigr/src).

The result is [cute_utf](https://github.com/RandyGaul/cute_headers/blob/master/cute_utf.h), yet another single file header library. Perfect for doing any string processing stuff, and not overly complicated in any way. Just do a quick google search for utf8 string libraries and every single one of them is absolutely nuts. They are all over-engineered and heavyweight. Probably because they all attempt to be general-purpose string libraries, which are debatably dumb to begin with.

The hard part of localization is probably the rendering of all kinds of glyphs. Rendering aside, localization can be as simple and storing original English translations in text files. A localizer (translator) can read the English and swap out all English phrases for utf8 encoded text symbols by typing the phrases in their native language. As long as localization is planned from project start, it can be very easy to support!
