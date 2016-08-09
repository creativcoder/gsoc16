## Project Overview

Implementing Service Worker infrastructure in Servo Browser Engine.

This project aims to implement the foundations necessary for the Service Workers Api in [Servo Browser Engine](https://github.com/servo/servo). The implementation as of now is a non-standard implementation, which does not mandate the use of promises as the promises api hasn't landed in Servo yet, also the cache and notification api is not implemented due to same reason.

* [Servo Project Wiki](https://github.com/servo/servo/wiki/Summer-of-Code-2016:-ServiceWorker-infrastructure)
* [Mozilla Proposals Page](https://summerofcode.withgoogle.com/organizations/5256839985889280/#4504639135285248)

The end goal of the project is to make the basic service worker dom api's available on Servo browser, and to allow registration of Service Workers and to be able to make them run when navigated to that scope.

The following api's are available on servo to interact with Service Workers:

* ServiceWorkerContainer : i.e., `navigator.serviceWorker` on the navigator dom object.

* ServiceWorkerRegistration

* ServiceWorkerGlobalScope

* ServiceWorker

## Related Pull Requests

* [Interface for Custom Responses](https://github.com/servo/servo/pull/10961) Merged
* [Implement related Service Worker interface and Register method](https://github.com/servo/servo/pull/11114) Merged
* [Integrate Service worker manager thread](https://github.com/servo/servo/pull/11727) Merged
* [Remove expect calls in service worker manager thread](https://github.com/servo/servo/pull/12518) Merged
* [Bring back run_with_memory_reporting in serviceworkerglobalscope](https://github.com/servo/servo/pull/12557) Merged
* [Make the service worker send custom response](https://github.com/servo/servo/pull/12582) Merged
* [Dispatch lifecycle events to service worker object and refactor html tests](https://github.com/servo/servo/pull/12682) Merged

## Service Worker Design overview

* [Design_overview](design_overview/README.md)

## Meta Project Tracker

* [Implement Service Workers API](https://github.com/servo/servo/issues/11091)

## Unresolved Issues
* [Provide a Sender for ServiceWorker object to communicate to its ServiceWorkerGlobalScope](https://github.com/servo/servo/issues/12773)

## Test page for Servo specific Service Workers implementation

* [gh-pages](https://github.com/creativcoder/gsoc16/tree/gh-pages)

## Related Reads

* [Service Workers Spec](https://github.com/slightlyoff/ServiceWorker)
* [Servo Design Wiki](https://github.com/servo/servo/wiki/Design)

## Blog posts

* [Basic Service Worker support lands in Servo](http://creativcoder.xyz/post/service-worker-in-servo/)
* [Post community bonding period with Servo](http://creativcoder.xyz/post/post-community-bonding-gsoc-servo/)
* [Introduction to Service Workers](http://creativcoder.xyz/post/service-workers-on-web/)
* [GSoC begins with Servo](http://creativcoder.xyz/post/so-it-begins-gsoc16/)

## Talks presented during GSoC'16 period

* [Contributing to Servo](https://github.com/opensource101/contributing_to_servo)