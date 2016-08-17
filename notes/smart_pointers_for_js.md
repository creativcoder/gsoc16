Servo relies on SpiderMonkey's mark and sweep garbage collector for cleaning up unused DOM objects.
In order to express the intent of managing the lifetime of DOM objects, we have two types in Rust.

SpiderMonkey's garbage collection works in two phases: Mark and Sweep 

Initially the GC creates a tree of all the allocations (GC Things). All the allocations which are directly accessible constitute what we call a Root Set.

Mark Phase: The GC traverses the tree of allocations and starts marking all reachable nodes as live.
Sweep Phase: This phase de-allocates all the allocations which were not marked as live, and thus makes up space for further allocations.

Root<T> - This is a stack based reference. Its added to the Root Set of the GC, which means its directly reacheable.

JS<T> - Types wrapped around JS is available to be garbage collected.
