
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


- [X] - Hooking the interface's receiver side to ScriptThread's Event Loop. The script thread needs a seperate channel pair to receive the network side's sender, which needs to be created at the instantiation of the script thread. Then inside `handle_msg()`, we match on the received events, and if we get the the sender, we check whether the current document, is controlled by any active service worker. If it is, then we send a custom response, to the sender.

(Update)

A seperate channel was created under script_thread.rs, that will facilitate fetching an `IpcSender<Option<CustomResponse>>`, towards script_thread.rs, that will then be used to send, the a custom response. to network code, in case the document is controlled by a service worker, and the client opts in for a custom response, which is invoked inside the FetchEvent.

A handler function was added, under script thread

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

Over the network, side if we receive a Some(custom_response), then we send that response, instead the actual network response

```rust
let (msg_sender, msg_receiver) = ipc::channel().unwrap();
    match load_data.source {
        RequestSource::Window(ref sender) => {
            sender.send(msg_sender.clone()).unwrap();    
            let received_msg = msg_receiver.recv().unwrap();
            if let Some(custom_response) = received_msg {
                let metadata = Metadata::default(doc_url.clone());
                let readable_response = custom_response.to_readable_response();
                return Ok(StreamedResponse::custom_response(metadata, box readable_response));
            }
        }
        RequestSource::Worker(ref sender) => {
            sender.send(msg_sender.clone()).unwrap();    
            let received_msg = msg_receiver.recv().unwrap();
            if let Some(custom_response) = received_msg {
                let metadata = Metadata::default(doc_url.clone());
                let readable_response = custom_response.to_readable_response();
                return Ok(StreamedResponse::custom_response(metadata, box readable_response));
            }
        }
        RequestSource::None => {}
    }
```
