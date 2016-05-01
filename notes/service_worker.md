###### A Service Worker interface:

* Runs in its own global script context(its own thread).

* Isn't tied to a particular page.

* Has no DOM access.

* Can run without any page at all

* Can terminate when it isn't in use, and run again when needed (its event driven)

* Is HTTPS only

Api of Registration of Service Workers is as follows:

```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/app/sw.js', {
    scope: '/app/'
  }).then(function(reg) {
    console.log('Registered', reg);
  }).catch(function(err) {
    console.log('Error', err);
  });
}
```

Events to listen to inside service worker global scope

* Activation

* FetchEvent


A document will pick a service worker to be its controller when it navigates, so the document you called .register from isn’t being controlled, because there wasn’t a service worker there when it first loaded.

If we refresh the document, it’ll be under the service worker’s control. We can check navigator.serviceWorker.controller to see which service worker is in control, or null if there isn’t one.

###### Issue Notes

MetaBug Tracker for Service Workers for Gecko
[Bugzilla](https://bugzilla.mozilla.org/show_bug.cgi?id=903441)

Stopping Service Workers
[Chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=372673)

and some other links:

[ServiceWorker - Chromium](https://docs.google.com/document/d/1zM6H3WlwL411xJbEjsZb5pcMeFJsoLK1CQWVQlkIg4A/edit)

[ServiceWorker Discuss Forum](https://groups.google.com/a/chromium.org/forum/#!forum/service-worker-discuss)
