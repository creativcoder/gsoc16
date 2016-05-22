##Personal Details

Name: Rahul Sharma
Email: rsconceptx@gmail.com
IRC nick: `creativcoder` on irc.mozilla.org
Telephone: +91-7687953404, +91-9661157908
Other contact methods: Github | Twitter | Linkedin
Country of residence: India
Time Zone : IST (UTC+05:30)
Primary language: English
###Project
Implementing Service Worker infrastructure in Servo Browser Engine
Mentor:Josh Matthews

**Abstract**

Servo is a new parallel browser engine being developed by Mozilla using a type safe language called Rust. Further design decisions of Servo are described here. Existing standard dom apis are being added to servo, from a lot of contributors around the world, to progress it further towards a production ready browser.

Context:  As of now, Servo has support for Web Workers, and the new Fetch Api whose development is in progress, is another new standard, that aims to replace XMLHttpRequest (XHR), and provides a cleaner api, over the uneasy syntax of XHR’s. Fetch Api, aims to provide a unifying interface, on the web, for anything involving requests and responses.The Fetch api spec also has a subsection which details on handling fetch calls within a service worker context.

Service Workers is a new specification, for modern web apps, which aims to bring native app like experience to browser based apps, by providing a programmable api, for caching certain paths of pages, intercepting network requests according to user logic. Service Workers are a derivative of Web Workers, that aren’t tied to any particular page, and can be shared on several browsing contexts. It has its own, `ServiceWorkerGlobalScope` execution context, that contains an instance of the active Service Worker. Spec

Servo has a partial implementation of Fetch Api, but the Service Worker Api’s are currently missing and if implemented, Servo will be able to take advantage of features that they provide like, intercepting network calls made from within the Service Worker’s context. For that the “handle_fetch” subsection of the Fetch Spec, is to be implemented. Spec




###Proposal
My project is to have a partial implementation of Service Worker Api and its helper interfaces in Servo Browser Engine. The project is focused on working on parts of the Service Worker Specification which revolves around intercepting network requests.

*The goals of the project*:

- Implementation of Service Workers API, will enable Servo to register and run a Service  worker script against a specified scope(url).
- Provide an execution environment for the Service Workers to run against a specified url.
- Intercept requests the user makes from a Service Worker controlled page, by handling the FetchEvent, within their scope. For a given request if the current page is under the scope of any active Service Worker we should be able to send custom response back to the user.
- One such use cases with this can be to provide a fallback response to user when internet connectivity is not available, or to re-route requests to another path in the origin.
- To share existing service worker instances under different browsing contexts.

Areas that this project does not cover are:

- The sub-steps dealing with `promises` within mentioned interfaces will not be implemented as Servo currently does not have native support of them.
 - The Cache Api.
 - Push Notification Api.


I plan on implementing relevant parts from the following interfaces of the Service Worker Spec:
Full Spec here

**Client Context**:
*ServiceWorker* - The actual interface that embodies the service worker.
*ServiceWorkerRegistration* - This interface is used to hold the mapping of service workers with their scopes (url).
*ServiceWorkerContainer* - It facilitates registering of service workers.This interface is being exposed on the `navigator` object, and will be used to register service workers, against the specified scope url.

**Execution Context**:
*ServiceWorkerGlobalScope* - This interface provides the Service workers their own execution context for the actual service worker script, and contains the event loop, with its task queues.
*FetchEvent* Interface [spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#fetch-event-section).
`respondWith` method will be used to send controlled response back to fetch caller - [spec](https://slightlyoff.github.io/ServiceWorker/spec/service_worker/#fetch-event-respondwith)

Further Development plans after completing GSOC project : Continue working towards implementing the missing spec parts when promises arrive, expand the work to Caching and Push notifications.


###Schedule of Deliverables

The whole of GSOC coding period is roughly 12 Weeks, and i have planned accordingly:

It appears that my exams are from 23 May to 3 June, so during this period, i will only be able to give around 2-3 hours to the project on exam days and 5-7 hours on weekends, and because of that, I plan to start early on my project, appropriately right after proposal selections are announced or even before that. I have made the work period of each module, to roughly 2 weeks, which will cover writing tests, getting code review and documentation for it. I intend to keep in sync about my progress with mentor every 2-3 days, and do pull requests in small chunks as per interfaces.

####Timeline

**Before May 1**: Discuss in thorough detail with mentor and other knowledgeable team members about Service Worker Api, about the way it is to be exposed to the user agent, and decide workarounds for required parts which involve promises, and a unit test strategy using testharness-js. Start with some dummy service workers interfaces and test locally to get used to the codebase. Also get more details about event loops and script thread and channels in the parts involving ServiceWorkerGlobalScope, during this time.

**May 1-14**: As a prerequisite for network fetch events, we need an interface around networking code, (i.e., the net/http_loader.rs and net/fetch/methods.rs) to be able to intercept network requests, and allow custom response back to the user.
This can be done by attaching a field (which represents the source from where the request originated) to the `LoadData` struct being sent to http_loader’s factory method’s closure.
Then we can set up a trait, that will have a method to return response, which can be implemented by `WrappedHttpResponse` struct.
We can do a match on that property, and accordingly alter the logic from the http_loader and fetch/methods, to send a custom response using the above implemented trait method.
Further implementation details, will be formalized upon implementation and discussion with mentor.
This will be used by network requests which are initiated from `script/` side.
Write unit tests for the above under “tests/unit/net”. Write docs, and make pull requests for above.

**May 15-28**  My Semester Exams will start by last week of this time.
Start with ServiceWorker and ServiceWorkerRegistration interfaces as these will form the basis of further development down the roadmap.
For ServiceWorker: We write respective rust code, from the web IDL definitions and bindings.
Add initialization methods for it, and enum types for its states.
Implement getters for `id`, `scriptURL`,and `active` attributes.

Start work on a partial implementation of ServiceWorkerRegistration enough to return instances of Service worker on its attributes such as `controller`, ‘active’, `waiting`,`scopeUrl`.  Write respective rust code from the WebIDL spec.


We can then test the Service Worker by exposing it to the dom, with reflect_dom_object with initializer stubs, using devtools, and then write a new set of tests for it using “./mach create-wpt” and testharness.js framework. Make pull request separately for them.


**May 29- June 11**:  If needed, we can implement the ServiceWorkerContainer and expose it to navigator object’s `serviceWorker` property. 
Implement `register` method for it, which returns an instance of ServiceWorkerRegistration, created in previous week.
Implement EnvironmentSettingsObject i.e.,(ServiceWorkerClient), which will be useful under service worker context. This can be implemented as a struct, whose instance will be available to be used by ServiceWorkerGlobalScope.
Write tests for above, get code review, write documentation, make pull requests for above.


**June 12 - June 20**:  Start working on ServiceWorkerGlobalScope: The code from the dedicatedworkerglobalscope.rs, can be used as a reference in its implementation.
Write rust code for the generated webIDL bindings from the codegen,
Prepare stubs for `onfetch`, and `onmessage` handlers.
We also need an environment settings object i.e, a ServiceWorkerClient, or we can use properties from the Global object.
Implement Run a Service Worker within the SerivceWorkerGlobalScope instance from spec. 
Create wpt unit tests for them, create documentation. Prepare codes for mid-term evaluation.


**June 21-July 3**: - Resume work on remaining parts of `ServiceWorkerRegistration` interface.
If needed, We also need an abstraction of a Job, which can be implemented inside the ServiceWorkerRegistration object as a struct.
Wire up logic for sharing service workers instances that are already, active at a given context. This can be done by querying the active ServiceWorkerRegistration for any active workers, and if so, we will use that worker for the any given new path on the origin. [Match Service Worker Registration] Algorithm.
Add other algorithms, required such as `getRegistration`.
Write unit tests for wpt, which will test whether service workers are getting initialized and stored under the registration objects, and other cases like getting the active worker. Write documentation for above followed by pull request.

**July 4-July 17**:  Start working on FetchEvent object which is required for “onfetch” event inside ServiceWorkerGlobalScope context.
If needed, We can implement the ExtendableEvent interface, which all Service Workers depend on, for lifecycle and functional events, otherwise only extend from Event.
Create basic FetchEvent construct which extends ExtendableEvent or Event as implemented above. This event will only be available in `serviceworkerglobalscope`.
Add attribute methods for `request`, `clientID`,’isReload’.


**July 18-31**: - Continue with FetchEvent,
Add necessary stubs for onfetch handler in ServiceWorkerGlobalScope. (Handle Fetch algorithm).  `dedicatedworkerglobalscope.rs` ‘handle_event’ method can be used as a reference to its implementation.
Write unit tests, under wpt tests, and make pull requests for the above.


**Aug 1 - Aug 14**:  Integrate FetchEvent object created above with the onfetch handle_event, dispatched on the ServiceWorkerGlobalScope.

Implement `respondWith` method which is called inside the active service worker. Inside the FetchEvent event object, we will call the Handle Fetch Algorithm to pass the repsonse to Fetch api. Write wpt unit tests for them, prepare documentation, followed by pull request.
Work on improving the test suites for both net/ and script/dom, considering cases for the above implemented interfaces. 
Prepare codes for final evaluation.

**Aug-15-Aug 23**: This is mostly kept as a buffer to complete any remaining work, and if time permits, work on above parts that can be made better. Additionaly if time permits i can work on
MessageEventInterface - Used by service workers to receive messages and post messages.
Terminating an active Service Worker.


Work Experience / Open Source Development

* December (2014) (2nd Year) - Android Intern at Sybrant Technologies, Chennai. Helped the android team, in integrating google maps api, in one of their in house application.

* November (2015) (3rd Year) - Android Intern Bufo Innovations (Startup), Kolkata. Developed a prototype, application based automobile ticket system under smart city project of Kolkata.

* Cloud Computing Workshop (March 2015) - (Cloud) - Organized a cloud computing workshop for junior students, on basics of cloud computing and setting up a private cloud infrastructure using Eucalyptus framework. Workshop_resources.


Hobby projects:

CodinCloud : (Work in Progress)-  A dev workspace on cloud. Idea is to create a similar platform like cloud9 IDE.  (An alpha demo CodinCloud available on Heroku PaaS)

Silica: (Work in progress) - A dead simple static site generator being written in Rust.Uses docopt standards for command line parsing and supports watching file changes to recompile markdown pages. Silica in future will employ web component based themes, and style for managing static sites.

Tweeto: Tweeto was an attempt to create a twitter like site using, using Meteor Js isomorphic reactive  javascript framework, with basic support for registration, follow and following feature, and ability to show time of tweets in realtime.

Academics

I am currently a 3rd Year undergraduate Computer Science Student
at Supreme Knowledge Foundation Group Of Institutions , Kolkata. India.
Topics which i am attracted to are programming languages, formal languages, paradigms, functional reactive programming, realtime web technologies.

####Why Me

I truly believe in open source software, and to me open source information matters a lot over the internet, and it matches with my interest on Working with Mozilla.

I have a couple of commits to Servo and other rust related contributions, [PR's](https://github.com/pulls?q=is%3Apr+author%3Acreativcoder+is%3Aclosed).
I also participated in [Rust DoSelect Hackathon](https://doselect.com/hackathon/doselect-rust-easy-hackathon) mainly as an exercise for learning rust. I was able to solve 5 out of 6 problems. Solutions on [Github](https://github.com/creativcoder/Rustacian/tree/master/doselect_hackathon_solutions).
I remain active on my Github, doing side projects and poking around with new and upcoming technologies.
I have a blog [creativcoder: random musings on bits and bytes](http://creativcoder.xyz/)
I answer questions about Rust and other computer science topics at Quora,[Rahul on Quora](https://www.quora.com/profile/Rahul-Sharma-556).

Development Tools: Arch Linux | i3 dm  |  Vim or Sublime Editor
Frameworks and Platforms: Flask, MeteorJS, Nodejs, Android.
Familiarity with Cloud PaaS : Heroku
Languages: C/C++, Python, Javascript, Rust.


Why Mozilla

Mozilla has always been and encourages others to be open. Today’s Internet stands upon, free information and i think, working with mozilla this GSOC, will mark an important role in my career. I will not only be contributing to mozilla, but the internet as a whole by this project, and that thing excites me the most. The quote from Mozilla is a great motivation - `We’re building a better Internet`.

Browser Engines are a complex piece of software and they do not get made every day. They are the most used software on a daily basis, so developing for a part of Servo browser engine will, help me appreciate how a browser engine works, and with Rust as a modern and safe language, i will be able to appreciate modern software development practices.
