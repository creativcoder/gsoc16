
##### ServiceWorkerRegistration

This interface, holds active, installing and waiting service workers.

It is consulted, whenever a navigation or a fetch, event occurs, 
to get the active service worker and act accordingly.

Service Worker Registration match should be done in longest-prefix-match way.

Currently if there're multiple registrations for the same origin we naively return the first matching one.

[Issue](https://bugs.chromium.org/p/chromium/issues/detail?id=370773)

From discussion with Josh, 

The registration maps, need to be stored under thread local storage, per script thread 
which is created for each document which gets created.

The script thread, exposes a `set_registration` associated method, which can be provided the scope url
and the registration dom object.
