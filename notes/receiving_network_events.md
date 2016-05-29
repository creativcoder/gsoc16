
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
```

So we spawn an event loop under serviceworkercontainer and start listening for network event.

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
                    Some(sender) => {sender.send(200).unwrap();},
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
