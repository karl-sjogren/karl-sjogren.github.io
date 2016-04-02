---
title:  "Lazy loading external scripts in Ember"
date:   2015-08-05 11:43:00
categories: ember javascript
---
When building a large Ember application you might get to a point when you feel that the initial download size is just to large. In my current project we got almost 2mb of javascript when all the templates and extra utilities are compiled and bundled. This is for an internal function so 95% of the time speed won't be a problem but as developers we strive to deliver a great experience for all our users.

Looking through our dependencies in `bower.json` I quickly found two huge offenders. We are utilizing [TinyMCE](http://www.tinymce.com/) as base for our "WYSIWYG"-editor and [vis.js](http://visjs.org/) to render a nice timeline. Both these adds a few hundred kilobytes each to our `vendor.js` that need to be downloaded initially for all first time users.

So how can we get around adding these dependencies to `vendor.js` but still use them in our application? This turned out to be quite easy by simply adding them to the DOM in the `model`-hook using a promise. I created two methods, one for js and one for css, and put them in a small utility script which can be seen below.

**app/utils/external-resource-loader.js**

{% highlight js %}
import Ember from 'ember';

export var loadScriptResource = function loadScriptResource(uniqueName, path) {
    return new Ember.RSVP.Promise(function(resolve, reject) {
      if(!!document.getElementById(uniqueName)) {
        return resolve();
      }

      var element = document.createElement('script');
      element.id = uniqueName;
      element.src = path;
      element.addEventListener('load', function() {
        resolve();
      });
      element.addEventListener('error', function() {
        reject(`Failed to load ${uniqueName} (${path})`);
      });
      document.getElementsByTagName('head')[0].appendChild(element);
    }, `Loading external script resource ${uniqueName} (${path})`);
};

export var loadStyleResource = function loadStyleResource(uniqueName, path) {
    return new Ember.RSVP.Promise(function(resolve, reject) {
      if(!!document.getElementById(uniqueName)) {
        return resolve();
      }

      var element = document.createElement('link');
      element.id = uniqueName;
      element.href = path;
      element.rel = 'stylesheet';
      element.addEventListener('load', function() {
        resolve();
      });
      element.addEventListener('error', function() {
        reject(`Failed to load ${uniqueName} (${path})`);
      });
      document.getElementsByTagName('head')[0].appendChild(element);
    }, `Loading external style resource ${uniqueName} (${path})`);
};
{% endhighlight %}

Then in the route where you need the script your simply import the methods and set them up as parts of your route model.

**app/routes/sauce.js**

{% highlight js %}
import Ember from 'ember';
import { loadScriptResource, loadStyleResource } from 'sauces/utils/external-resource-loader'

export default Ember.Route.extend({
  model: function(params) {
    var visjs = loadScriptResource('visjs', '/assets/visjs/vis.min.js');
    var viscss = loadStyleResource('viscss', '/assets/visjs/vis.min.css');

    return Ember.RSVP.hash({
      awesomeSauces: this.get('store').findAll('sauce')
      visjs: visjs,
      viscss: viscss
    });
  }
}
{% endhighlight %}
The name you give the script when loading it will be used to identify the script once loaded so it won't be loaded again so you can reuse the same name across several routes and it will still only be loaded once.

This change saved us about 600 kilobytes on the initial load, and for users who never watch the timeline [vis.js](http://visjs.org/) will never be loaded at all.