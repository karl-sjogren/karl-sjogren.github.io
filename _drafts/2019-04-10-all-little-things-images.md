---
title:      "All the little things - Images"
date:       2019-04-10 12:00:00
categories: saga javascript service-workers performance
---

One of my personal goals within Saga has always been to get responses out fast
and with as small payload as possible. A Saga site presents a lot of images,
from editorial images to cover images for books, so optimizing this have been
a huge benefit.

First of all, serving images from a secondary domain was a given. No cookies
saves us about 1kb per request which builds up a lot over time. Just the start
page over at [Familjen Helsingborg](https://www.bibliotekfh.se/) currently loads
25 images, so thats 25kb of cookies not sent. Seeing that only five of the
images are actually over 25kb (and those are for the large carousel) this is
quite a save.

### Content negotiation

Another important aspect to lower the amount of data sent is to send the best
image to each client. There are two important signals here, the `Accept`-header
and [Client hints](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints).

The `Accept`-header lets use know which image format the current browser is
accepting, this helps us selecting if we should serve an ordinary JPEG, JPEG XR
or WEBP. Again using the start page of [Familjen Helsingborg](https://www.bibliotekfh.se/)
as our sample, Chrome (WEBP) loads 506kb of images, Edge (regular JPEG) loads
650kb and then IE (JPEG-XR) loads 700kb (!). Yeah there is something wrong with
that last one. But clearly browsers supporting WEBP has an edge here.

Client Hints is currently only supported in
[Chromium-based browsers](https://caniuse.com/#search=client%20hints) but for
the mobile market that is still quite a win. What Client Hints does is that they
tell the browser to append information about the current viewport size to the
image requests. In Saga we use this to limit the size of images sent to the
browser by always limiting the width to the viewport width.

...

{% highlight javascript %}
var configurationUrl = '/api/configuration';
var defaultCoverUrl = '/covers/default';
var customCoverUrl = '/api/configuration/custom-cover';
var defaultImageData = null;
var contentUrls = [];

var isCoverRequest = url => {
  var valid = false;
  contentUrls.forEach(contentUrl => {
   if(url.includes(contentUrl) && url.includes('/covers/')) {
     valid = true;
   }
  });
  return valid;
};

var getDefaultCover = () => {
  var coverUrl = contentUrls[0] + defaultCoverUrl;
  return fetch(customCoverUrl)
  .then(response => {
    return response.blob();
  }, () => {
    return fetch(coverUrl).then(response => {
      return response.blob();
    }, () => {
      console.error('Failed to retrieve any default cover. The ServiceWorker will be quite useless.')
      return null;
    });
  });
};

self.addEventListener('install', (/* event */) => {
  self.skipWaiting();
});

self.addEventListener('activate', event => {
  var promise = fetch(configurationUrl).then((response) => {
    return response.json();
  }).then((configuration) => {
    contentUrls = configuration.contentUrls;
    console.info('ServiceWorker will intercept the following urls.', contentUrls);

    return getDefaultCover();
  }).then((image) => {
    defaultImageData = image;
  }, () => {
    defaultImageData = null;
    return Promise.reject('Failed to load the default cover. Rejecting promise and disabling interception.');
  });
  event.waitUntil(promise);
});

self.addEventListener('fetch', event => {
  var promise;
  if(!isCoverRequest(event.request.url) || defaultImageData === null) {
    promise = fetch(event.request);
  } else {
    var fetchRequest = event.request.clone();
    var url = fetchRequest.url;
    if(url.includes('?')) {
      url += '&no-default=true';
    } else {
      url += '?no-default=true';
    }

    fetchRequest = new Request(url, {
      method: fetchRequest.method,
      mode: 'cors',
      credentials: 'omit',
      redirect: fetchRequest.redirect,
      headers: fetchRequest.headers
    });

    promise = fetch(fetchRequest).then(response => {
      if(!!response && response.status === 200) {
        return response;
      }

      var fallback = new Response(defaultImageData, {
        ok: response.ok,
        status: response.status,
        url: url
      });

      return fallback;
    }).catch(error => {
      console.warn('Request failed', error);
    });
  }

  event.respondWith(promise);
});
{% endhighlight %}

### Summary

...