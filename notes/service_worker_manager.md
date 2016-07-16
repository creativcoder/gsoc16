
To manage running service workers, at different origins we need a Service Worker Manager, which lets us keep track of all registered service workers as well as any active service workers.

The Service Worker Manager is instantiated in script::init(), during the constellation creation process.

Update 01 June 2016

Some discussions over irc.

Cases we need to handle:
1) same-origin request matches active service worker, 
2) cross-origin request matches active service worker,
3) same-origin request matches inactive service worker,
4) cross-origin request matches inactive service worker

Entities outside the script thread, must not store JS<T> types as they 
do not have a reference to their runtime, in which JS objects live in.

Trait objects must not be sent cross process.

Upon, calling `register()` from service worker container, the script thread is instructed to store the scope url and the ScopeThings struct in its TLS.

When constellation receives the navigation events, then it notifies the script thread, along with a (sender to the service worker manager), that 
script thread can use to send the activator closure to the manager, so that it call call the closure to spawn the respective service worker.

The SW manager's need to be one per content process and one per constellation to be able to send a sender to it from each script thread
that gets created.
