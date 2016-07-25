
NOTE: The code snippets mentioned in these notes, may not reflect the current implementation in Servo, as different design descisions
had to be taken further down the road. But this provides a history of changes of how it was approached.

This week's goals: (Merged in [PR](https://github.com/servo/servo/pull/10961))

### Prerequisites for Fetch Event Handling in the `components/net`

##### Interfaces for Intercepting Network Requests under, Window and Worker Scope.

- [X] - Basic Interface for routing custom response, from client to network code(http_loader). Inside `net_traits/lib.rs` we have setup an enum,

```rust

pub type CustomResponseSender = IpcSender<Option<CustomResponse>>;
pub type CustomResponseReceiver = IpcReceiver<Option<CustomResponse>>;

#[derive(Clone, Deserialize, Serialize, HeapSizeOf)]
pub enum RequestSource {
    Window(#[ignore_heap_size_of = "Defined in Std"] IpcSender<CustomResponseSender>),
    Worker(#[ignore_heap_size_of = "Defined in Std"] IpcSender<CustomResponseSender>),
    None
}

```

Which serves to provide a wrapper for client requests (the `LoadData`) that are controlled by service workers to allow to send an Optional CustomResponse to the network code.


- [X] - Hooking the interface's receiver side to ScriptThread's Event Loop. The script thread needs a seperate channel pair to receive the network side's sender, which needs to be created at the instantiation of the script thread. Then inside `handle_msg()`, we match on the received events, and if we get the the sender, we check whether the current document, is controlled by any active service worker. If it is, then we send a custom response, to the sender.

(Update 04/05/16)

A seperate channel was created under script_thread.rs, that will facilitate fetching an `IpcSender<Option<CustomResponse>>`, towards `script_thread.rs`, that will then be used to send, the a custom response. to network code, in case the document is controlled by a service worker, and the client opts in for a custom response, which is invoked inside the FetchEvent.

A handler function was added, under script thread, and here's an example of sending a mock response, to the network code.

```rust
fn handle_custom_msg(&self, msg: IpcSender<Option<CustomResponse>>) {
        // detect controlling service workers here
        let mock_response = CustomResponse::new(
        Headers::new(),
        RawStatus(200, Cow::Borrowed("Ok")),
        b"<h1>Servo</h1><h3>The Browser Engine from Future</h3>".to_vec()
    );  
    	// assuming our document is controlled by a service worker,
    	// and that it had initiated a load request, with a custom response
    	// so we send that response to http_loader.rs
        msg.send(Some(mock_response)).unwrap();
    }
```

Over the network side (`components/net/http_loader.rs`), if we receive a custom_response from the client if it sends one, then we send that response instead of the real http response.

```rust
let (msg_sender, msg_receiver) = ipc::channel().unwrap();
    match load_data.source {
        RequestSource::Window(ref sender) | RequestSource::Worker(ref sender) => {
            sender.send(msg_sender.clone()).unwrap();
            let received_msg = msg_receiver.recv().unwrap();
            if let Some(custom_response) = received_msg {
                let metadata = Metadata::default(doc_url.clone());
                let readable_response = to_readable_response(custom_response);
                return StreamedResponse::from_http_response(box readable_response, metadata);
            }
        }
        RequestSource::None => {}
    }
```

The Pending loads initiated from the `document.rs`, also need to be tracked upon, so we also add a `source` field to `PendingAsyncLoad`, to intercept network requests from that as well. Similarly the `xmlhttprequest.rs` also needs to be modified accordingly under its `Send` method.

(Update 09/05/16)

The `LoadData` constructor was modified to, take a trait object which represents common routines for knowing things about a request as:

```rust

pub trait LoadOrigin {
    fn referrer_url(&self) -> Option<Url>;
    fn referrer_policy(&self) -> Option<ReferrerPolicy>;
    fn request_source(&self) -> RequestSource;
    fn pipeline_id(&self) -> Option<PipelineId>;
}

```
