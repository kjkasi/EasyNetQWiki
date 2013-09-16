A list of changes to be done for EasyNetQ version 1. **Warning**: most are breaking changes to the current API.

* ~~[5] Factor exchange, queue, binding declares out of AdvancedBus publish and subscribe.~~
* [5] Publish back on IBus interface. Marshall publish calls from multiple threads onto a single publish loop on a single publish channel.
* [3] Better request/response.
* [3] Request back on IBus interface. Similar to Publish.
* [5] Timeouts! Including for publisher confirms, request response.
* [3] Make publisher confirms the default for IBus.Publish.
* [3] Send/Receive pattern.
* [3] Subscriber cancellation.