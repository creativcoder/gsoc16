After a lot of redesign along with helpful guidance from Josh and awesome people at Servo we finally have a working implementation of Service Workers (hereby mentioned as SW) capable enough to monitor network requests made from different origins and optionally send back custom responses on a registered origin.

This post will give an overview of the current implementation of SW's in Servo. Please note that the codebase in future is likely to face several changes, due to interdependence over several features such the Fetch Api [Tracking Issue](https://github.com/servo/servo/issues/11894), whose implementation is in progress and the Promise API [Tracking Issue](https://github.com/servo/servo/issues/4282).

To get a basic idea of what Service Workers are refer to my [previous](http://creativcoder.xyz/post/service-workers-on-web/) blog post about them.

Before we talk about them, some basic constructs in Servo's architecture needs to be mentioned.
This only covers topics in a way that is in interest to SW's implementation. To get a broad overview of Servo's architecture refer, to the Wiki or my brief blog post.

A constellation is a co-ordinator of various pipelines.
A pipeline in Servo is an abstraction of a Script Thread, Layout Thread, Paint Thread.
We'll only focus on Script Thread here.
The Script Thread is the one that holds the DOM data structure for a page loaded in Servo browser instance.

Service workers are designed so as not to depend to any context/Document and so they must be at a higher level in the abstraction hierarchy than the Script Thread. They can listen to page load requests from different requests. As such a service worker manager (hereby mentioned as SWM) thread was devised to manage different service worker registrations at different origins. The idea is to have a single instance manager that can store information of all registered SW's and can listen to requests of activation or timeouts of any of the SW's.

* The init phase:

![alt init_phase](assets/init_phase.jpg)

The constellation thread is the one that's created before any of the above mentioned threads are created. So it creates required channels for SWM, and when the constellation creates the script thread, the SWM's sender is passed to it, which spawns the SWM thread. Also during constellation initialization

* The registration phase:

![alt init_phase](assets/reg_phase.jpg)

Lets take an example of a page `https://example.com` where we register a SW over `https://example.com/profile` scope.
From the implemented DOM interfaces we register a Service Worker by calling `navigator.serviceWorker.register('sw.js', {scope: '/profile'});` which returns a registration object. This registration object is then stored in the current ScriptThread's thread local storage.
The worker does not start its scope when registered; instead it forwards the required attributes to the SWM, which controls the starting and stopping of ServiceWorkerGlobalScopes.

* The network phase:

![alt network_phase](assets/network_phase.jpg)

When the browser is navigated to the url `https://example.com/profile` where the SW was registered The `http_loader`, sends a sender to the SWM, and asks whether the current load url has a running service worker that may have a custom response. Here the SWM, forwards the network's sender, to the running ServiceWorkerGlobalScope's event loop, which then is able to reply with any custom response that is cached in.

TODO: improve this document
