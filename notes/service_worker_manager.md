To avoid, running service worker scopes forever, we can use the script threads event loop, to hook any activation requests
originated by navigate events in constellation. 

We introduce a:

A service worker manager, to marshall activation requests from the constellation navigate events.
and send it back to the script thread, so it can invoke it from the per script thread registration map, stored in it.

// manages various service workers running at different origins in the document
struct ServiceWorkerManager {
 	workers: HashMap<Url, Box<Fn()>>,
 	// the receiver to receive ServiceWorkerMsg's from constellation
 	receiver: IpcReceiver<ServiceWorkerMsg>
}

 impl ServiceWorkerManager {
 	fn new(receiver: IpcReceiver<ServiceWorkerMsg>) -> ServiceWorkerManager {
 		ServiceWorkerManager {
 			workers: HashMap::new(),
 			receiver: receiver
 		}
 	}
 	fn start(&mut self) {

 		while let Ok(msg) = self.receiver.recv() {
 			match msg {
 				ServiceWorkerMsg::ActivateOnScope(url, script_sender) => {
 					//workers.get(&url).activate();
 					debug!("activation for scope : {:?}", url)
 					script_sender.send(ActivateServiceWorker(url.clone())).unwrap();
 				}
 				ServiceWorkerMsg::NewRegistration(url) => {
 					//let registration = ScriptThread::get_registration(&url);
 					//self.workers.insert(url.clone(), registration.get_activator());
 					debug!("registration for scope : {:?}", url)
 				}
 				ServiceWorkerMsg::Exit => {break;}
 			}
 		} 
 	}
 }

impl ServiceWorkerMessenger for IpcSender<ServiceWorkerMsg> {
    fn new() -> IpcSender<ServiceWorkerMsg> {
        let (sender, receiver) = ipc::channel().unwrap();
        spawn_named("ServiceWorkerManager".to_owned(), move || {
            ServiceWorkerManager::new(receiver).start();
        });
        sender
    }
}

pub trait ServiceWorkerMessenger {
    fn new() -> Self;
}

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

Upon, calling `register()` from service worker container, the script thread, is instructed to store the scope url and the Boxed closure to its,
local storage.

When constellation receives the navigation events, then it notifies the script thread, along with a (sender to the service worker manager), that 
script thread can use to send the activator closure to the manager, so that it call call the closure to spawn the respective service worker.
