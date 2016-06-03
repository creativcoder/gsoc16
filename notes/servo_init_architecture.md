
Overview:

The browser engine, starts its lifecycle, by creating the [Compositor](https://github.com/servo/servo/blob/master/components/servo/lib.rs#L114) followed by the [Constellation](https://github.com/servo/servo/blob/master/components/servo/lib.rs#L154), as part of the window, initialzation procedure.

Constellation:
The constellation, creation process involves, the creation of all the required senders, for threads such as resource thread,
image cache thread, bluetooth thread, font thread; as the initial constellation state. 

Pipeline: A pipeline is an abstraction of a process, encapsulating a a script thread, a layout thread

Initial Page load and pipeline creation: 
[handle_init_load](https://github.com/servo/servo/blob/master/components/constellation/constellation.rs#L917) is called, just after creation of constellation, as cause of sending it a `IntiLoadUrl` message. At this stage, the first pipeline is created, `new_pipeline()` which in turn calls Pipeline::spawn, which spawns a new content-process (basically a page in browser parlance) comprising of three threads, `PaintThread`, `ScriptThread`, `LayoutThread`, which holds all states, of that page.

The `Pipeline` struct is a uniquely identifiable, entity, that holds information about the constent-process.

The constellation maintains, all the pipeline instance created in its `child_processes` field.
