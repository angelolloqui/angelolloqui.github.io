---
layout: post
title:  "Data Races with value types in Swift"
date:   2022-08-26 12:00:00
categories: 
    - ios
    - swift
    - datarace
    - bugs
permalink: /blog/:title
---

This week I had an [interesting discussion around a possible data race condition](https://github.com/GetStream/swift-activity-feed/issues/19) due to wrong threading synchronisation when manipulating a value type (a `String` in this case) in a class. 

## The buggy code
```
final class MyClass {
    var token: String
    
    init(_ token: String = "") {
        self.token = token
    }

    func myMethod() -> Bool {
        token.isEmpty
    }
}
```
In a first look, this might seem correct. We have just a `var` with a `String`, which is a value type, and a method to just check if the `token` is empty that only calls the `isEmpty` from `String`. Straightforward code and safe right? well, it is OK as long as you do not introduce threading, but the moment you do it will not. Let me elaborate.

## The test

If you run this test with Thread Sanitizer enabled:

```
 func test_data_race() {
        let sut = MyClass()

        DispatchQueue.concurrentPerform(iterations: 1_000_000) { i in
            sut.token = "\(i)"
            _ = sut.myMethod()
        }
    }
```

you will see this output:
```
WARNING: ThreadSanitizer: data race (pid=8329)
  Read of size 8 at 0x000107c1aab8 by thread T2:
    #0 closure #1 in DataTests.test_data_race() DataTests.swift:69 (Tests:arm64+0xde354)
    #1 partial apply for closure #1 in DataTests.test_data_race() <compiler-generated> (Tests:arm64+0xde3e4)
    #2 partial apply for thunk for @callee_guaranteed (@unowned Int) -> () <null>:73675156 (libswiftDispatch.dylib:arm64+0x42f4)
    #3 _dispatch_client_callout2 <null>:73675156 (libdispatch.dylib:arm64+0x35dc)

  Previous write of size 8 at 0x000107c1aab8 by main thread:
    #0 closure #1 in DataTests.test_data_race() DataTests.swift:69 (Tests:arm64+0xde374)
    #1 partial apply for closure #1 in DataTests.test_data_race() <compiler-generated> (Tests:arm64+0xde3e4)
    #2 partial apply for thunk for @callee_guaranteed (@unowned Int) -> () <null>:73675156 (libswiftDispatch.dylib:arm64+0x42f4)
    #3 _dispatch_client_callout2 <null>:73675156 (libdispatch.dylib:arm64+0x35dc)
    #4 _swift_dispatch_apply_current <null>:73675156 (libswiftDispatch.dylib:arm64+0x43a0)
    #5 @objc DataTests.test_data_race() <compiler-generated> (Tests:arm64+0xde448)
    #6 __invoking___ <null>:73675156 (CoreFoundation:arm64+0x11c5ec)

  Location is heap block of size 32 at 0x000107c1aaa0 allocated by main thread:
    #0 __sanitizer_mz_malloc <null>:73675156 (libclang_rt.tsan_iossim_dynamic.dylib:arm64+0x51004)
    #1 _malloc_zone_malloc <null>:73675156 (libsystem_malloc.dylib:arm64+0x1527c)
    #2 DataTests.test_data_race() DataTests.swift:66 (Tests:arm64+0xde07c)
    #3 @objc DataTests.test_data_race() <compiler-generated> (Tests:arm64+0xde448)
    #4 __invoking___ <null>:73675156 (CoreFoundation:arm64+0x11c5ec)

  Thread T2 (tid=6246748, running) is a GCD worker thread

SUMMARY: ThreadSanitizer: data race DataTests.swift:69 in closure #1 in DataTests.test_data_race()
```
So ThreadSanitizer is detecting a **data race in the code when accessing the token**. 
What does it mean? basically you are making a wrong usage of the variable. It gets read and write operations concurrently but the variable itself is not protected, and the fact that it is a value type does not help.

What can this cause? it is undefined, but in practice most likely you will have a crash when compilation optimizations are enabled.

## The fix
OK, so this simple code can crash when reading and writing the token in parallel from different threads! How can we fix it? we just need to make serial access to read/write. There are multiple ways of doing it (with different primitives), but this could be one:

```
final class MyClass {
    private let syncQueue = DispatchQueue(label: "com.test.myQueue", attributes: .concurrent)
    private var _token: String
    var token: String {
        get {
            syncQueue.sync {
                _token
            }
        }
        set {
            syncQueue.async(flags: .barrier) {
                _token = newValue
            }
        }
    }

    init(_ token: String = "") {
        _token = token
    }

    func myMethod() -> Bool {
        token.isEmpty
    }
}
```
As you can see, what we did is to protect the `var` by forcing serial writing to it, so multiple reading can happen but only 1 thread can execute a write at a time (the barrier waits for all previous readings to finish and postpones all subsequent read/write accesses till the write is done). The resulting code is slower to execute, but it is now safe.


## Final thoughts
I wanted to share a few thoughts around this issue, that are common misconceptions in the Swift community:

### ❌ Value types are thread safe
Since the value type has a copy semantic, it may seem logical to think that they are inherently protected from data races. However, that is not the case. **Swift does not guarantee thread safety in value types**, so accessing any `var` from multiple threads is a potential data race condition. This issue of course does not apply to `let` variables since they are immutable.

### ❌ Value types are always copied
That is the semantic but not really what happens under the hood. When passing value types around, Swift compiler is smart enough to know if the copy is needed, removing unnecessary copies. In practice it uses a CopyOnWrite(COW) strategy, where it will **make the copy only when the value is modified**, but not when passed around. As a result, in most situations you will actually have a pointer to the same underlaying memory address even when using value types.

### ❌ Tests do always behave like production code
The fact that a test does not crash is no guarantee to assert that some code can not crash in production. Tests run in simulated environments and **they normally have different compilation options than the ones in your final builds**. For example, ARC will make aggressive optimizations when compiling with the proper options, so lots of unnecessary retain/releases will be removed from final builds. In this particular case, my test suit was not crashing, and I was only able to see some wrong usage by activating the Thread Sanitizer.

## Extra reading
- [Understanding Swift's value types thread safety](https://forums.swift.org/t/understanding-swifts-value-type-thread-safety/41406)
- [ARC optimizations](http://angelolloqui.com/blog/21-ARC-I-Introduction-to-ARC-and-how-it-works-internally)


