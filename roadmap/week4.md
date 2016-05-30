
This weeks goals:

- [X] Create interfaces for Constellation, to communicate a navigate event, or a network load event. to the
ServiceWorkerContainer's event loop.

- [X] Implement Activate Event mechanism, on serviceworkerglobalscope.

- [ ] Implement the FetchEvent interface. 

In the current implementation the ServiceWorkerContainer will act as a broker for getting the mentioned events.

An approach towards the api can be something like:

The service worker container will, spawn a thread upon initialization, for listening to events from the window "load" or "navigation" events.

```rust

enum NetworkEvent {
  LoadEvent(LoadData),
  NavigateEvent(LoadData),
}
```

The contellation will send the serviceworkercontainer's, event loop, a navigate event, by using the received sender,
which was passed to it when register is called.

The constellation keeps a map of registered service workers, keyed by their scope url.

The serviceworkercontainer upon receiving the event from constellation,

1) It will check whether the active worker's (i.e., serviceWorker.controller), registration's scope url, falls under the received url's path fragment from load_data, and if it matches it will dispatch the fetch event on the serviceWorker.controller.

2) If not, it will hand over the received url from load_data to match_registration() ([Match Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#scope-match-algorithm)), if a registration is found then it will spawn the service worker thread, switch its state to active, and dispatch the fetch event on it.

From discussion with Josh,

The registration map for the storing all registrations per origin, should be stored in each script thread's local storage, to avoid duplicating
the regisration objects.
