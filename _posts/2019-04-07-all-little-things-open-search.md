---
title:      "All the little things - OpenSearch"
date:       2019-04-07 12:00:00
categories: all-the-little-things opensearch c# aspnetcore saga json-ld schema.org
---

Since a huge thing in Saga is its search engine, adding an OpenSearch endpoint
was a given. This might seem like a small thing and the average internet user
won't know what it is but a lot of people appreciate it being there.

So what does it get you? Well first of all it is a standardized way to declare
how to search your website which is always a good thing since this makes your
site easier to understand for browers and other services.

Then there is the user facing parts. In Firefox you get the option to add a new
search provider when visiting a site with OpenSearch.

![Add search provider in Firefox]({{ site.url }}/assets/add-search-provider.png)

After adding the search provider, typing a search query into the address field
will allow you to search directly on the site by using the icons at the bottom
of the dropdown.

![Execute a search provider in Firefox]({{ site.url }}/assets/execute-search-provider.png)

Chrome goes even further and automatically loads search providers when visting
a site for the first time. It will then be integrated into what they call the
"[Omnibox](https://www.chromium.org/user-experience/omnibox)" so that it can be
used quickly with only the keyboard.

![Chrome Omnibox]({{ site.url }}/assets/chrome-omnibox.gif)

So as we see this is something that we really want to have, especially if we have
a site where searching is a huge part of the reason to visit it.

### Implementing the OpenSearch endpoint

If adding this to a regular website it would be enough to just put a static xml
file somewhere on the site and linking it in the header, but for a CMS like Saga
we of course needed to generate this dynamically. Since we are using ASP.Net Core
in Saga the easiest way to do it was using a controller action.

{% highlight csharp %}
public class OpenSearchController : Controller {
    private readonly ISiteSettings _siteSettings;

    public OpenSearchController(ISiteSettings settings) {
        _settings = settings;
    }

    [HttpGet("open-search")]
    public ActionResult Status() {
        var xml =
            $@"<OpenSearchDescription xmlns=""http://a9.com/-/spec/opensearch/1.1/"" xmlns:moz=""http://www.mozilla.org/2006/browser/search/"">
<ShortName>{WebUtility.HtmlEncode(_settings.SiteName)}</ShortName>
<Description></Description>
<InputEncoding>UTF-8</InputEncoding>
<Image height=""64"" width=""64"" type=""image/png"">{_settings.SiteUrl}/images/favicon.png</Image>
<Url type=""text/html"" method=""get"" template=""{_settings.SiteUrl}/search?query={{searchTerms}}"" />
<Url type=""self"" template=""{_settings.SiteUrl}/open-search"" />
<moz:SearchForm>{_settings.SiteUrl}/search</moz:SearchForm>
</OpenSearchDescription>";

        return Content(xml, "application/opensearchdescription+xml");
    }
}
{% endhighlight %}

While this could have been done by building an XDocument instead, which would
be safer, this was a quick fix early in development that has worked fine since
then.

So then we just need to wire it up with a link-tag in the header template.

{% highlight html %}
<link rel="search" type="application/opensearchdescription+xml" href="/open-search" title="The Local Library" />
{% endhighlight %}

And then we are done! Next time you visit the site the OpenSearch stuff will be
available.

### Going the extra mile

So while OpenSearch is great, [structured data](https://developers.google.com/search/docs/guides/sd-policies)
was also something we used quite a bit in Saga. I might talk more about that in
a later post but for now, the website search snippet is the interesting part.

It is as simple as adding this snippet to your layout template.

{% highlight html %}
<script type="application/ld+json">
{
  "potentialAction": {
    "@type": "SearchAction",
    "query-input": "required name=search_term_string",
    "target": "https://www.example.com/search?query={search_term_string}"
  },
  "@context": "http://schema.org",
  "@type": "WebSite",
  "url": "https://www.example.com"
}
</script>
{% endhighlight %}

### Summary

So this is just one of the small tings we did with to make Saga a great CMS. I've
had the chance to ask both our customers and their users about this and while
most doesn't seem to know about all this, they are all both surprised and excited
when they find out about it.