
This week's goals:

- [X] Prepare a [test_bed](https://github.com/creativcoder/gsoc16/tree/gh-pages) for testing service workers init phase and running phase.

- [X] Implement timeout notification to service workers manager on active running service workers.
 
 Debug and test it. 

The sel.wait() in receive_events() in `serviceworkerglobalscope`, is a blocking call so, we need to instead rely on a timer_port on the,
workerglobalscope to take action for timeout events from workers.

- [ ] Re-iterate code, on matching of scope urls of registered service workers. As per spec, matching needs to happen in, longest prefix way.
