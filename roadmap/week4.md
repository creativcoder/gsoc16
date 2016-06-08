
This week's goals:

- [X] Create interfaces for Constellation, to communicate a navigate event, or a network load event. to the
ServiceWorkerContainer's event loop.

- [X] Implement Activate Event mechanism, on serviceworkerglobalscope.

- [X] Integrate the service worker manager thread in the init phase of content-process and constellation initialization


The current implementation as of now has a `ServiceWorkerManager`, which is the co-ordinator, of all running service workers, requested to  be started by any script_thread.

This `ServiceWorkerManager` has to have a single instance, for the initial launch of servo.

The `script::init()` can be an ideal place where we can initialize it and pass on its sender to the contellation
The constellation saves this sender.

When navigations occur, the constellation sends the navigate message to script thread, the script thread, then checks for its local registration map, and if it finds a registration that matches the scope, it sends the message back to constellation, with the serializable entities that `ServiceWorkerManager`, will be able to spawn a worker from them.

TODO: Test message relay under different cases of page navigation, .i.e., under iframes.
