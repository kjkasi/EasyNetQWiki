A list of changes to be done for EasyNetQ version 1. **Warning**: most are breaking changes to the current API.

* Factor exchange, queue, binding declares out of AdvancedBus publish and subscribe.
* Publish back on IBus interface. Marshall publish calls from multiple threads onto a single publish loop on a single publish channel.
* Better request/response.
* Request back on IBus interface. Similar to Publish.
* Timeouts! Including for publisher confirms, request response.
* Make publisher confirms the default for IBus.Publish.
* Send/Receive pattern. 