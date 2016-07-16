
This week goals:

- [X] Eliminate need of sending constellation navigate message to script thread, instead route, the scope thing back to constellation, as it should be given the responsibility to talk to service worker manager

- [X] Rethink design of passing a service worker object over to manager.

* The manager will store the `ScopeThings`, right at the invocation of `navigator.serviceWorker.register()`.

* The `ScopeThings` struct is being passed to service worker manager via `handle_serviceworker_registration`, which stores it.
When navigation happens, and the network sender, expects a optional custom repsonse,
the script thread, will do a match on the url for which the navigation is requested.

* If a scope matches in a longest prefix way, we send that scope url which the manager will use to get the
scope things and spawn the respective service worker for that object.
