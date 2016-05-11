
#### ServiceWorkerContainer

This interface provides the ability to register serviceworker, through the `navigator.serviceWorker` property
on the navigator interface. `navigator.serviceWorker` returns an instance of ServiceWorkerContainer, which
has a register method available that, takes in a service worker script url,
which when successful, returns a service worker registration object, holding the active service worker.

For current implementation, an approach can be like this:

when `register()`, is invoked, a new `ServiceWorker` object will be created, which, in turn will call the,
`run_serviceworker_scope()`.

Then a new `ServiceWorkerRegistration`, object is created, whose `active`, field is set to the ServiceWorker
instance created above.

Then we return the registration object.
