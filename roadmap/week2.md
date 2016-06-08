
This week's goals: Merged in [PR](https://github.com/servo/servo/pull/11114)

Implement the Building blocks for Service Workers

(Interfaces)

- [X] - ServiceWorker - The interface which represents a service worker, contains state information and methods to communicate to the document via postMessage.
- [X] - ServiceWorkerContainer (focused on the Register Method) - Provides facilities to register the service worker, currently has only the register method implemented.
- [X] - ServiceWorkerRegistration basic interface - Holds the service worker registration and their scope url, which they intend to control.
- [X] - ServiceWorkerGlobalScope - holds the execution environment of a Service worker in execution.

ServiceWorker: 

- [X] - State - returns the state fo the service worker, this is currently being set as `Activated`, when `init_service_worker` is called.
- [X] - ScriptUrl - returns the url of the script resource that it contains.

ServiceWorkerContainer:

- [X] - GetController - returns the active controller, (i.e.,the activated service worker), and its being set when Registration of script happens.
- [X] - Register - Add more error handling to script Urls, and improve tests. Fix a bug of scope urls not being matched, even when provided
					from the webidl dictionary RegistrationOptions.


(Update) - May 19 2016

The [PR]() Consists of the related interfaces required for basic, working of the service worker against a specified scope, and the ability to maintain a registration map, and ability to instantiate the registration of the service workers.

In [PR](https://github.com/servo/servo/pull/11114), ServiceWorkerContainer interface's controller attribute's manipulation was facilitated by making available a trait based method, on it, and passing it to the registration interface, so that it may set the controller attribute from that side.
