
This weeks goals: 

- [ ] Integrate Service Worker Client with the interfaces built in previous week.

Service Worker Client represents a browsing context or a document which is currently browsing,
the pages, and which may be possibly controlled by any registered service worker.

[Client](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#client)

The Client will represent the window global object, or it can represent an worker global object.

The client object can be used to, listen for navigation events or network events from, so that appropriate
event can be sent to the active service worker.

From the spec, the Client interface is to be instantiated, at the time of creating the serviceworkerglobalscope, 
of service workers.


- [ ] Decide on the location of persisting of the registration maps. This would probably be under service workercontainer instance,
as its alive for the duration of the window object.

- [ ] Improve wpt-tests for registration object and scope urls.

- [ ] Create constructs for matching and getting the registration objects from the registration map.
