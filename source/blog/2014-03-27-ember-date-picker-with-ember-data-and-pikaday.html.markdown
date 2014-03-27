---
title: Ember Date Picker with Ember Data and Pikaday
date: 2014-03-27 01:55 UTC
tags: ember,ember-data
---

[Atomic Spin has a great article][1] on setting up [Pikaday][3] with an `Ember.TextField` view.
However, since I'm using Ember Data with some `date` attributes, I needed to actually use a `Date` object rather than a formatted date string. Here's what I came up with after adding a few things to their example:

<div>
  <a class="jsbin-embed" href="http://emberjs.jsbin.com/gumex/1/embed?html,js,output">Ember Starter Kit</a><script src="http://static.jsbin.com/js/embed.js"></script>
</div>

---

There were a couple things I had to add for this to work:

- I'm using the [moment.js][2] library to allow `MM/DD/YYYY` formatting.
- Rather than binding the attribute directly to the `value`, I'm using a `date` binding instead. This gives me the chance to parse a formatted date before setting the attribute.
- I added the `init` function so I can format the existing date when the form loads.
- The `updateDate` function, which observes `value`, directly sets the `date` to the parsed value.

[1]: http://spin.atomicobject.com/2013/10/29/ember-js-date-picker/
[2]: http://momentjs.com/
[3]: https://github.com/dbushell/Pikaday
