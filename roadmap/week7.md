
This weeks goals:

This week has largely been a lot of refactors to the existing code.

[X] Resolve code review on [#11727](https://github.com/servo/servo/pull/11727) and improvements to the ServiceWorkerManager code and its interaction with constellation.

[X] Remove the need for `RequestSource`; as the constellation itself has the information of network load events.

[X] Refactor the matching of scope in the Service Worker Manager.

Issues to resolve:

* Bring back the memory profiler hooks inside the serviceworkerglobalscope, as we don't have any available service worker instance as of now, in the serviceworkerglobalscope.

* Extract out fields from WorkerGlobalScopeInit, which can be re-used in the ServiceWorkerManager thread.