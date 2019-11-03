---
title:      "Creating LESS mixins from Fontello config"
date:       2019-11-03 12:00:00
categories: fontello javascript less node
---

[Fontello](http://fontello.com/) is a great service for managing icon fonts for
a project. For those who haven't used it it allows one to cherry pick icons from
a range of different font colletions into your own custom font. It also makes it
really easy to just upload your own SVG icons and include it in your custom font.

Creating your own custom font with just the icons needed has several advantages.
First of all it reduces the download size to just the needed glyphs (instead of
downloading the whole Font Awesome for example).

Secondly it also gives you a larger selection of icons, even though mixin a lot
of icon sets might not look that well. But not having a thousand generic icons
readily available also forces you to actually look for a good icon instead of
just taking something that is available. It might take a bit more time but in
the end it will hopefully lead to a better icon selection.

## Exporting from Fontello

When exporting from Fontello you get a zipfile with the following structure
(omitted most of the css files for brevity).

```
/config.json
/css/fontello.css
/demo.html
/font/fontello.*
/LICENSE.txt
/README.txt
```

Importing `fontello.less` (and probably updating the font paths) is all that
you need to get it up and running which is great for some projects. For larger
projects though we've always resorted to using mixins for icons and our designer
manually updated our mixins everytime we added a new icon. This did of course
increase the time to add a new icon by a lot.

So per usual, when someone is doing something manually it can be automated if
they just ask. And within a development team there is usually someone who can
do it.

## Converting the Fontello config to a LESS mixin

Create a new file in your project called `fontello-to-less.js` and paste this
code into it. Adjust the part where we import the font config on line 3 if it
doesn't match up with your workspace.

{% highlight js %}
/* global require, module */
const fs = require('fs');
const config = require('../fonts/config');

// This method is used to read the fontello config and output
// the icons as Less mixins
const fontelloToLess = function fontelloToLess(outputPath) {
  const lineBreak = '\r\n';
  let output = '#icons {' + lineBreak;
  for(let glyph of config.glyphs) {
    output += `  .${glyph.css}() {` + lineBreak;
    output += `    content: '\\${glyph.code.toString(16)}';` + lineBreak;
    output += `  }` + lineBreak;
  }
  output += '}';

  fs.writeFileSync(outputPath, output);
}

module.exports = fontelloToLess;
{% endhighlight %}

Then either create a new file to invoke the task like below or adjust it to make
it a part of your build pipeline.

For a simple invokation via node it could look like this (lets call this file
`build.js`).

{% highlight js %}
const fontelloToLess = require('./fontello-to-less');
fontelloToLess('./styles/fontello-icon-definitions.less');
{% endhighlight %}

Then call it like `node build.js`.

This will output a LESS file that looks like this in `./styles/fontello-icon-definitions.less`.

```
#icons {
  .arrow-left() {
    content: '\e800';
  }
}
```

Which makes it a lot easier to work with the icons if you ask me. I've so far
used this with two projects, one with a simple Grunt pipeline and with with a
Ember CLI pipeline and I'll probably use this with a lot of future projects as
well.