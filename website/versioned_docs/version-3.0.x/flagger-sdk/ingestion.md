---
id: version-3.0.x-ingestion
title: Ingestion
sidebar_label: Ingestion
original_id: ingestion
---

`Flagger` automatically collects every "decision" that was made(for example, whether flagIsEnabled true or false for a 
given entity), groups this decisions up in "ingestion requests" and then sends it to the Airdeploy.

In the nutshell ingestion is the process of sending collected data to the server. 
This process is fully automatic and requires only one thing from the developer - to gracefully shutdown `Flagger SDK` at 
the end of the application runtime:

<!--DOCUSAURUS_CODE_TABS-->
<!--Javascript-->
```javascript
await Flagger.shutdown() // shutdown is a Promise
```
<!--Ruby-->
```ruby
Flagger.shutdown(3000)
```
<!--Python-->
```python
Flagger.shutdown(3000)
```
<!--Go-->
```go
Flagger.shutdown(3000)
```
<!--Java-->
```java
Flagger.shutdown(3000)
```
<!--Swift-->
```swift
// iOS
func applicationWillTerminate(_ application: UIApplication) {
    _ = Flagger.shutdown(timeoutMillis: 1000)
}

// anywhere else:
_ = Flagger.shutdown(timeoutMillis: 1000)
```
<!--END_DOCUSAURUS_CODE_TABS-->

> Note: It's important to call shutdown before the end of the runtime to prevent data loss.