
##### Notes on ServiceWorkerRegistration

Service Worker Registration match should be done in longest-prefix-match way.

Currently if there're multiple registrations for the same origin we naively return the first matching one.

[Issue](https://bugs.chromium.org/p/chromium/issues/detail?id=370773)
