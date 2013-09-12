---
title: Speeding up Rails 4 in Development Mode
date: 2013-09-12 17:00 UTC
tags: rails
---

As I've been adding more and more CSS/JS files to my Rails app, I've been
noticing a huge slowdown in page load times. Most of the assets are tiny
controller-specific files, but the size of those files is irrelevant.

By default, Rails 4.0 turns on "debug mode" for your assets. As a result, every
single page load sends a separate request for each asset. Even though most of
these return a 304 Not Modified header, it still slows things down by having to
send/receive a couple hundred requests.

Simply disabling asset debugging in `config/development.rb` drastically
improved the page load time in the browser:

```ruby
# Debug mode disables concatenation and preprocessing of assets.
# This option may cause significant delays in view rendering with a large
# number of complex assets.
config.assets.debug = false
```
Note that this is different from production mode; Rails still
recognizes when you change your assets and will automatically reload them
without needing to restart the server. The primary change is that the
CSS/JS is concatenated into single files.

Before the change, Chrome took around 1.8 seconds to fully load a page. After
the change, 450ms. Development mode feels __much__ snappier now!
