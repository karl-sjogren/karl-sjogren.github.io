---
title:      "The great Saga"
date:       2019-04-07 06:00:00
categories: all-the-little-things saga
---

Saga is a product that started development mere weeks after I joined Open
Library Solutions back in 2013. We were a small time, at no time more then four
developers and one designer but for the most part just two and a half devloper.
The core team have been me and two colleagues since the start

The goal was to create a new and more modern version of the companys older
product CSLibrary, a website with integrated library catalog search and other
regular CMS functions to publish content.

It took a few weeks of evaluating different content management systems to base
it on. We made a huge list of features that we had in the old product that we
needed to be able to support. Below are some of the more complex ones.

* Multiple languages within the same site.
* Some sort of widgets or user controls with support for advanced editors (ie
  not just property grids).
* Time-based content publishing (ie I want a new version of this page to be put
  online tomorrow).
* Easily customizable, at least on the public website part, but still easy to
  upgrade to a new version.

In the end we settled for a completely custom solution, most systems we evaluated
either didnt't have all the features we needed or was way to hard to customize and
then upgrade.

Now, six years later, we have a fully custom CMS tailored to our exact needs. We
have advanced widgets, with entirely customizable editor UI; we support multiple
languages and have connections between pages in different languages; we have full
control to publish content at certain times and also archiving it at a later time;
etc. The admin interface as well as the API is part of the product deployment
while each custom website is its own git repository that is just merged with
each new release when upgrading.

The name Saga was chosen after a year or so of development. We went through a
bunch of names but felt that Saga was a good fit since it works across languages,
it can be related to libraries and it just sounds good.

It took us almost three years to get our first customer up and running (with a
small detour of a couple of months for another project) but in the end we
delivered something amazing. And now another three years later there are hundreds
of library across Sweden, Norway and Finland using Saga.

![Familjen Helsingborg running on Saga 3.0]({{ site.url }}/assets/bibliotekfh.png)

Above is the site for [Familjen Helsingborg](https://www.bibliotekfh.se/), a
collaboration of 11 municipalities in Sweden running on Saga 3.0. The main layout
of the website is similar to a standard Saga site but they have some own things
going on as well. It is one of the more advanced customers as of writing and they
have some really good editors, the site might turn up from time to time in other
posts as an example.