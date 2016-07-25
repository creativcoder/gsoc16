
NOTE: The code snippets mentioned in these notes, may not reflect the current implementation in Servo, as different design descisions
had to be taken further down the road. But this still provides a history of changes of how it was approached.

This week's goals: Merged in [PR](https://github.com/servo/servo/pull/11114)

- [X] Integrate Service Worker Client with the interfaces built in previous week.

Service Worker Client represents a browsing context or a document which is currently browsing,
the pages, and which may be possibly controlled by any registered service worker.

[Client](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#client)

The Client will represent the window global object, or it can represent an worker global object.

The client object can be used to, listen for navigation events or network events from, so that appropriate
event can be sent to the active service worker.

From the spec, the Client interface is to be instantiated, at the time of creating the serviceworkerglobalscope, 
of service workers.


- [X] Decide on the location of persisting of the registration maps. This would probably be under serviceworkercontainer instance,
as its alive for the duration of the window object.

- [X] Create constructs for matching and getting the registration objects from the registration map.

- [X] Improve wpt-tests for registration object and scope urls.

Update (May 21 2016)

Experiment with hooking the event handling for fetch events; as of now a basic `ServiceWorkerFetchHandler` struct has been implementation which can be used by the FetchEvent interface, to handle events related to network load.

```rust

pub struct ServiceWorkerFetchHandler {
    addr: TrustedServiceWorkerAddress,
    request: LoadData 
}

impl ServiceWorkerFetchHandler {
    pub fn new(addr: TrustedServiceWorkerAddress, request: LoadData) -> ServiceWorkerFetchHandler {
        ServiceWorkerFetchHandler {
            addr: addr,
            request: request
        }
    }
} 

impl Runnable for ServiceWorkerFetchHandler {
    fn handler(self: Box<ServiceWorkerFetchHandler>) {
        let this = *self;
        ServiceWorker::handle_fetch(this.addr, this.request);
    }
}

```

