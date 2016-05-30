
From discussion with Josh;

Servo's constellation is in charge of sending load events from documents to script thread. To enable service workers to receive global wide network events, we also need to send, the message to, any active workers that may have been registered. by the any document.

So an approach would be to introduce a:

`HashMap<Url, IpcSender<NetworkEvent>>` as a field in constellation, so that it can store service worker descriptors keyed by their scope.

Under `serviceworkercontainer.rs`

let (container_sender, container_receiver) = ipc::channel().unwrap();
        self.global().r().constellation_chan().send(ScriptMsg::NewServiceWorker(scope.clone(), container_sender.clone()));

        // A event loop to receive any load or navigate events from constellation
        spawn_named("ServiceWorkerContainerEventThread".to_owned(), move || {
            loop {
                match container_receiver.try_recv() {
                    Ok(val) => {/*do stuff*/},
                    Err(e) => {}
                }
            }
        });

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


To Resolve: 

The current implementation, of the service worker event loop, always stays running, when registered.

```rust
loop {
                match from_constellation_port.try_recv() {
                    Ok(load_msg) => {
                        match load_msg {
                            FromScriptMsg::LoadUrl(pipeline_id, load_data) => {
                            debug!("load msg from {:?} {:?}",pipeline_id, load_data.url);
                            },
                            _ => {}
                        }
                        scope.mem_profiler_chan().run_with_memory_reporting(|| {
                        // Service workers are time limited
                        let sw_lifetime = Instant::now();
                        let sw_lifetime_timeout = prefs::get_pref("dom.serviceworker.timeout_seconds").as_u64().unwrap();
                            while let Ok(event) = global_for_event.receive_event() {
                                if scope.is_closing() {
                                    break;
                                }
                                global_for_event.handle_event(event);
                                if sw_lifetime.elapsed().as_secs() == sw_lifetime_timeout {
                                    break;
                                }
                            }
                        }, reporter_name.clone(), parent_sender.clone(), CommonScriptMsg::CollectReports);
                    },
                    Err(_) => {/*Loop again for next event from constellation */}
                }
            }
```

The ideal behaviour from the spec, should be to be able to call `activate_serviceworker_scope` at will, whenever
we actually have any navigation which is in interest to the registered service worker.
There needs to be a service worker manager thread, that should be able to spawn service workers, upon
receiving events from constellation. The api, of activate_serviceworker_scope should be such that it does,
not depend on GlobalRef, but instead must be provisioned required entities, as individual entities.
There needs to be a mediation, from the script thread, between constellation and the service worker manager thread.
