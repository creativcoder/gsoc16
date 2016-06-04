
From discussion with Josh;

Servo's constellation is in charge of sending load events from documents to script thread. To enable service workers to receive global wide network events, we also need to send, the message to, any active workers that may have been registered. by the any document.

So an approach would be to introduce a:

`HashMap<Url, IpcSender<NetworkEvent>>` as a field in constellation, so that it can store service worker descriptors keyed by their scope.

Under `serviceworkercontainer.rs`
```rust
        let (to_constellation_sender, event_from_constellation) = ipc::channel().unwrap();
        // these sender pairs are used to forward messages, to the sw-global scope, so that it becomes active from the
        // dormant state.
        let (scope_sender, scope_receiver) = ipc::channel().unwrap();
        global_ref.constellation_chan().send(ScriptMsg::ServiceWorkerDescriptor(scope.clone(), to_constellation_sender.clone()));
        // This listens for events from constellation
        activate_serviceworker(global_ref, &*active_worker, script_url.clone(), closing, sender, receiver, scope_receiver);
        thread::spawn(move || {
            loop {
                match event_from_constellation.try_recv() {
                    Ok(status) => scope_sender.send(status).unwrap(),
                    Err(_) => {}
                }
            }
        });
```.

Over constellation side,

we match on the script thread msg, and receive an event variant as: 

```rust
Request::Script(FromScriptMsg::NewServiceWorker(scope, container_sender)) => {
                self.handle_new_serviceworker(scope, container_sender);
                //container_sender.send(NetworkEvent).unwrap();
            }
```
and insert the sw-descriptor `into active_service_worker` Hashmap.

```rust

fn handle_new_serviceworker(&mut self, scope: Url, container_sender: IpcSender<NetworEvent>) {
        self.active_service_workers.insert(scope, container_sender);
    }
```

Then under main load url events we can have something like:

```rust
Request::Script(FromScriptMsg::LoadUrl(source_id, load_data)) => {
                debug!("constellation got URL load message from script");
                match self.active_service_workers.get(&load_data.url) {
                    Some(sender) => {sender.send(NetworkEvent).unwrap();},
                    None => {}
                }
                self.handle_load_url_msg(source_id, load_data);
            }
```

(Update 28 May 2016)

Over few iterations on the design of receiving events, the ideal approach can be:

The register method should be given following capabilites apart from steps mentioned in the spec:
- install the service worker - creating the ServiceWorker object.
- creating a registration object, then store the previously created service worker inside it.
- spawn an event loop, which can listen global network events, from the constellation
- match on the receive events and query the script thread's local registration map
- if we match on the registration, then call `run_serviceworker_scope`

The ideal behaviour from the spec, should be to be able to call `activate_serviceworker_scope` at will, whenever
we actually have any navigation which is in interest to the registered service worker.
There needs to be a service worker manager thread, that should be able to spawn service workers, upon
receiving events from constellation. The api, of activate_serviceworker_scope should be such that it does,
not depend on GlobalRef, but instead must be provisioned required entities, as individual entities.
There needs to be a mediation, from the script thread, between constellation and the service worker manager thread.

The contellation will send the service worker manager, event loop, a navigate event, by using the received sender.

The constellation keeps a map of registered service workers, keyed by their scope url.

The script_thread upon receiving the event from constellation,

1) It will check whether the active worker's (i.e., serviceWorker.controller), registration's scope url, falls under the received url's path fragment from load_data, and if it matches it will dispatch the fetch event on the serviceWorker.controller.

2) If not, it will hand over the received url from load_data to match_registration() ([Match Algorithm](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#scope-match-algorithm)), if a registration is found then it will spawn the service worker thread, switch its state to active, and dispatch the fetch event on it.

From discussion with Josh,

The registration map for the storing all registrations per origin, should be stored in each script thread's local storage, to avoid duplicating
the regisration objects.
