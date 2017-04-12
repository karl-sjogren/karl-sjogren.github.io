---
title: "Pre-encoded scripts and styles with ASP.NET Core and Gulp"
date: 2016-04-02 20:02:00
categories: misc aspnet netcore gulp brotli
---

Making websites fast to load has always been something I've kept close to my heart. Be it by
creating the correct indicies in a database, optimizing and caching frequently used data or
doing as much as we can with as few bytes as possible delivered to the client.

Today we will look at the last one, delivering optimized resources (in this case scripts and
styles) to the client. We will setup a small [Gulp](http://gulpjs.com/) script that minifies
our resources and then also compresses them with both broli and gzip so that we can return the
smallest possible files to the clients.

TL;DR Negotiated compression for gzip and brotli is awesome, sample can be found over at
[karl-sjogren/aspnet-resource-manifest-sample](https://github.com/karl-sjogren/aspnet-resource-manifest-sample).

## What is Brotli?

Brotli is a new and exciting compression algorithm that was first used for webfonts with the WOFF
2.0 format. It has later been made available for all types of content and can currently be used by
all major browsers except Safari. See [Can I Use on Broli Encoding](http://caniuse.com/#feat=brotli)
for up to date statistics.

css bytes https://bibliotek.jonkoping.se/css/saga-5fad3036c6.css.br
br: 20 019
gz: 24 588
minified: 179 311

js bytes https://bibliotek.jonkoping.se/js/saga-4a452ddada.js
br: 29 906
gz: 35 996
minified: 188 491

## Lets get started

We'll be using ASP.NET Core and Gulp 
<script src="~/js/site.min.js" asp-append-version="true"></script>

<script src="~/js/site.min.js"></script>