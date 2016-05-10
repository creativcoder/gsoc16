
This weeks goals: 

Implement the Building blocks for Service Workers

(Interfaces)

- [X] - ServiceWorker
- [X] - ServiceWorkerContainer (focused on the Register Method)
- [ ] - ServiceWorkerRegistration
- [X] - ServiceWorkerGlobalScope

ServiceWorker: 

- [X] - State
- [X] - ScriptUrl

ServiceWorkerContainer:

- [X] - GetController
- [X] - Register - Add more error handling to script Urls, and improve tests.


(Update) - May 10 2016

The [PR](https://github.com/servo/servo/pull/11114), adds a register method, which instantiates a new service
worker scope, creates a ServiceWorkerRegistration, and sets its active attr, to the created ServiceWorker

The [PR](https://github.com/servo/servo/pull/11114), does not update the ServiceWorkerContainer's controller attribute. This would involve a little api change, and an integration with a ServiceWorkerClient, which will be introduced by next week.
