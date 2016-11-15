
This weeks goals:

[X] Tidy up unused entities in ServiceWorker implementation.

[X] Provision a Sender to the ServiceWorker object to communicate with its corresponding serviceworkerglobalscope. [Issue](https://github.com/servo/servo/issues/12773)

[X] Decouple scope matching to a seperate method, and add senders list and message bufffers to Service Worker Manager.

[X] Implement `postMessage` on ServiceWorker object.

Issues to Resolve: 

Debug blocking issues and some of the messages not being relayed on the correct Service Worker.
