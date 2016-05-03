A Javascript engine begins its life with a JSRuntime, which is the parent object, for all things involving javascript. There is only a single instance of JSRuntime.

The JSContext object, is a derived or a child object of JSRuntime, that is actually responsible for executing js script code.
There can be many execution contexts, per thread created by the JSRuntime.

At the start, a global execution context is created. Its a container that can contain nested child execution contexts; whenever a new function scope is invoked in js code. It contains various runtime information such as variable name to value bindings.

The Environment Settings object, is basically all global information pertaining to the current runtime execution context, that child scripts (that are executed within the global execution context) can use, to find the information they need.

So, an environment settings object in the dom code, must be initialized and associated when we initialize instances of a window object, and a worker object, a shared worker, or a service worker object. These objects all have seperate execution contexts, so they will have different settings object.

Environment settings object should define the following fields, that can be queried by scripts having the same parent execution context:

* Script execution environment - basically the the parent execution context, holding the exection context stack, the global code, the function code and function objects. `JSContext`

* the global object of the script (defaults to window object inside a window) `GlobalRef`

* the browsing context associated with the object in context. `BrowsingContext`

* a reference to the event loop

* character URl encoding of the document (a utf-8)

* base url for api's called from the the script, that use this settings object. (document url)

* origin url
