Servo relies on SpiderMonkey's mark and sweep garbage collector, for cleaning up unused DOM objects.
In order to express the intent of managing the lifetime of DOM objects, we have two types in Rust.

SpiderMonkey's garbage collection works in two phases: Mark and Sweep

Initially the GC creates a tree of all the allocations, using stack reference of variables. All the the nodes referenced on the stack which are directly accessible

Mark Phase: 

Root<T> - This is a stack based reference. Its added to the Root Set of the GC, which means its directly reacheable.

JS<T> - This is available to be garbage collected.