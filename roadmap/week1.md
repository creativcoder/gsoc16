
### Prerequisites for Fetch Event Handling in the `components/net`

##### Interfaces for Intercepting Network Requests under, Window and Worker Scope.

- [X] - Basic Interface for routing custom response, from client to network code(http_loader). Inside `net_traits/lib.rs` we have setup an enum,

```rust

pub type CustomResponseSender = IpcSender<IpcSender<Option<CustomResponse>>>;
pub type CustomResponseReceiver = IpcReceiver<IpcReceiver<Option<CustomResponse>>>;

#[derive(Clone, Deserialize, Serialize, HeapSizeOf)]
pub enum RequestSource {
    Window(#[ignore_heap_size_of = "Defined in Std"] CustomResponseSender),
    Worker(#[ignore_heap_size_of = "Defined in Std"] CustomResponseSender),
    None
}

```

Which serves to provide, a wrapper for client requests (the `LoadData`) that are controlled by service workers, to allow to send an Optional CustomResponse, to the network code.


- [ ] - Hooking the interface's receiver side to ScriptThread's Event Loop. The script thread needs a seperate channel pair to receive the network side's sender, which needs to be created at the instantiation of the script thread. Then inside `handle_msg()`, we match on the received events, and if we get the the sender, we check whether the current document, is controlled by any active service worker. If it is, then we send a custom response, to the sender.

(Update)

A seperate channel was created under script_thread.rs, that will facilitate fetching an `IpcSender<Option<CustomResponse>>`, towards script_thread.rs, that will then be used to send, the a custom response. to network code, in case the document is controlled by a service worker, and the client opts in for a custom response, which is invoked inside the FetchEvent.