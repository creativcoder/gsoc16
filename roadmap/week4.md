
This weeks goals:

- [ ] Create interfaces for Script thread, to communicate a navigate event, or a network load event. to the
ServiceWorkerContainer's event loop.

- [ ] Implement Activate Event mechanism, on serviceworkerglobalscope.

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

For navigation, in window, handle_navigate() method, will send the serviceworkercontainer's, event loop, a navigate event.

The serviceworkercontainer upon receiving the event from window,

1) It will check whether the active worker's (i.e., serviceWorker.controller), registration's scope url, falls under the received url's path fragment from load_data, and if it matches it will dispatch the fetch event on the serviceWorker.controller.

2) If not, it will hand over the received url from load_data to match_registration() ([Match Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#scope-match-algorithm)), if a registration is found then it will spawn the service worker thread, switch its state to active, and dispatch the fetch event on it.