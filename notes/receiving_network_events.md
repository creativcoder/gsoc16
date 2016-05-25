
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
                    Some(sender) => {sender.send(200).unwrap();},
                    None => {}
                }
                self.handle_load_url_msg(source_id, load_data);
            }
```
