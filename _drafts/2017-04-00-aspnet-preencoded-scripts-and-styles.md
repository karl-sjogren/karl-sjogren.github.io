---
title: "Pre-encoded scripts and styles with ASP.NET Core and Gulp"
date: 2017-04-02 20:02:00
categories: misc aspnet netcore gulp brotli
---

Making websites fast to load has always been something I've kept close to my heart. Be it by
creating the correct indicies in a database, optimizing and caching frequently used data or
doing as much as we can with as few bytes as possible delivered to the client.

Today we will look at the last one, delivering optimized resources (in this case scripts and
styles) to the client. We will setup a small [Gulp](http://gulpjs.com/) script that minifies
our resources and then also compresses them with both broli and gzip so that we can return the
smallest possible files to the clients.

We'll be using ASP.NET Core and Gulp to get this up and running though the Gulp script could easily
be used with any language or framework.

TL;DR Negotiated compression for gzip and brotli is awesome and can save from 65-85%,
a working sample with ASP.NET Core can be found over at
[karl-sjogren/aspnet-resource-manifest-sample](https://github.com/karl-sjogren/aspnet-resource-manifest-sample).

## What is Brotli?

Brotli is a new and exciting compression algorithm that was first used for webfonts with the WOFF
2.0 format. It has later been made available for all types of content and can currently be used by
all major browsers except Safari. See [Can I Use on Broli Encoding](http://caniuse.com/#feat=brotli)
for up to date statistics. None of the browsers will send the correct accept-flag unless your site
is served over https or via localhost though.

## Prerequisites

Before we get started you should make sure that you have an up to date version of
[Node.js](https://nodejs.org/en/) as well the
[.NET Core 1.1 SDK](https://www.microsoft.com/net/download/core).

If you get compilation errors on the brotli packages later you might need to get
some extra tools installed that is needed for ``node-gyp`` to run properly, it
can be found over at their [GitHub page](https://github.com/nodejs/node-gyp).

## Lets get started

We'll start out with creating a new ASP.Net Core MVC site using ``dotnet new mvc``. This will create
a new basic MVC site to get us up and running. Then we'll need to intialize our ``package.json`` using
``npm init`` so we can install the needed packages.

As noted above, we'll use Gulp for this but we'll also need a few extra components, the below line
will install everything we need.

{% highlight shell %}
npm install gulp gulp-brotli gulp-clone gulp-gzip gulp-rev gulp-uglify gulp-less merge-stream --save-dev
{% endhighlight %}

So when the all this finishes we can finally start on some code! Create a new file called ``gulpfile.js``
in the root of your project and paste the following script into it.

{% highlight js %}
var gulp = require('gulp');
var less = require('gulp-less');
var uglify = require('gulp-uglify');
var rev = require('gulp-rev');
var gzip = require('gulp-gzip');
var clone = require('gulp-clone');
var brotli = require('gulp-brotli');
var merge = require('merge-stream');

gulp.task('default', ['scripts', 'styles']);

gulp.task('styles', function () {
  var base = gulp.src('wwwroot/css/**/!(*.min).css')
    .pipe(less({ compress: true }))
    .on('error', function(e) {
        console.error('>>> ERROR IN STYLES', e);
        this.emit('end');
      })
    .pipe(rev())
    .pipe(gulp.dest('wwwroot/css'));

  return preEncodeResources(base, 'wwwroot/css');
});

gulp.task('scripts', function () {
  var base = gulp.src('wwwroot/js/**/!(*.min).js')
    .pipe(uglify({ compress: true }))
    .on('error', function(e) {
        console.error('>>> ERROR IN SCRIPTS', e);
        this.emit('end');
      })
    .pipe(rev())
    .pipe(gulp.dest('wwwroot/js'));

  return preEncodeResources(base, 'wwwroot/js');
});

// Encodes the supplied stream using gzip and brotli and then
// appends the resulting filename into rev-manifest.json.
var preEncodeResources = function(baseStream, outPath) {
  var streams = [];

  // Clones the stream and compresses it using gzip with
  // level 9 (maximum) compression
  streams.push(baseStream
      .pipe(clone())
      .pipe(gzip({ append: true, gzipOptions: { level: 9 } }))
      .pipe(gulp.dest(outPath)));

  var MODE_GENERIC = 0,
      MODE_TEXT = 1;

  // Clones the stream and compresses it using brotli with
  // level 11 (maximum) compression
  streams.push(baseStream
      .pipe(clone())
      .pipe(brotli.compress({
        extension: 'br',
        skipLarger: true,
        mode: MODE_TEXT,
        quality: 11
      }))
      .pipe(gulp.dest(outPath)));

  // Writes the reved filename into the manifestfile
  streams.push(baseStream.pipe(rev.manifest('wwwroot/rev-manifest.json', {
      base: 'wwwroot/',
      merge: true
    }))
    .pipe(gulp.dest('wwwroot/')));

  // Merge the resulting streams so that we can return properly
  return merge.apply(null, streams);
};
{% endhighlight %}

This is a quite basic gulp script that compiles some LESS and then minifies some javascript,
the interesting parts doesn't start until the ``preEncodeResource`` method at the end. What
we want to do there is to use the same stream with three different targets, regular uncompressed,
gzipped and brotli encoded. To do this we'll use ``gulp-clone``to split the input stream into
three and then merge them together with ``merge-stream`` to get get a proper return value.

{% highlight html %}
<!-- Incorrect tag-->
<script src="~/js/site.min.js" asp-append-version="true"></script>
<!-- Correct tag -->
<script src="~/js/site.min.js"></script>
{% endhighlight %}

## What will we save?

I've compressed [jQuery 3.2.1](https://code.jquery.com/jquery-3.2.1.min.js) using this script
to get some reference values.

| Compression | jQuery 3.2.1       |
| ----------- | ------------------ |
| None        | 86 848 bytes       |
| GZip        | 30 352 bytes (34%) |
| Brotli      | 27 268 bytes (31%) |

Well that wasn't all that impressive, good but not impressive. But as with most compression
schemes you'll see larger gains on larger files and site with just jQuery won't be much. So
lets look at some more numbers. These were taken from a a site that we built at work for the
[libraries in Jönköping](https://bibliotek.jonkoping.se/) that uses an earlier version of this
script but with the same compression levels.

| Compression | CSS                | JS                 |
| ----------- | ------------------ | ------------------ |
| None        | 179 311 bytes      | 188 491 bytes      |
| GZip        | 24 588 bytes (13%) | 35 996 bytes (19%) |
| Brotli      | 20 019 bytes (11%) | 29 906 bytes (16%) |

That is a bit more impressive and actually worth putting some time into and with virtually no
performance hit on the server in the end.