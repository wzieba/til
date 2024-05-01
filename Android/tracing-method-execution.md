# Tracing method execution. Fixing binder performance bottleneck

Introduction to tracing and profiling: [Tracing 101](https://perfetto.dev/docs/tracing-101) from Perfetto documentation.

## `Debug.startMethodTracing`
The best way I've found so far is using - [android.os.Debug#startMethodTracing](https://href.li/?https://developer.android.com/studio/profile/generate-trace-logs) and opening the trace from Device Explorer (`sdcard/Android/data/<package>/files`) in Android Studio Profiler. **Important:** before opening a new trace, **close** the previous one. Double-click on a new trace won't close the previous trace (bug in AS). Realizing this took me longer than I'd like to admit.

Using this method I was able to identify a performance bottleneck in [Automattic-Tracks-Android](https://github.com/Automattic/Automattic-Tracks-Android) - an SDK we use for logging analytical events to Automattic service. I've executed:

```kotlin
Debug.startMethodTracing()  
    repeat(10) {  
        AnalyticsTracker.track(AnalyticsEvent.ORDERS_ADD_NEW)  
    }  
Debug.stopMethodTracing()
```

and it resulted with:

![image](https://github.com/wzieba/til/assets/5845095/60c5375a-7bc3-4924-9945-0947fc59e9ab)

Logging 10 events at the same time is not something that often happens, but it can happen. Freezing main thread for even half of this time, 100ms, is already bad enough to optimize it.

### Cause
Going deeper into trace tree, I've spotted that `android.os.BinderProxy.transact` takes significant amount if time. We use it to check for device properties (Bluetooth, Wifi, etc.). 

Reading [Android documentation](https://href.li/?https://developer.android.com/topic/performance/anrs/find-unresponsive-thread#consecutive-binder-calls) about ANRs, we can see a warning about avoiding "many consecutive binder calls" which happens here. Also, according to docs: "although most binder calls are quick, the long tail can be very slow.". We seem to experience the long tail, as we have recorded traces on Sentry showing 1.20s binder execution time:

![image](https://github.com/wzieba/til/assets/5845095/b871e4e8-ed44-434b-ab04-c3f4346ee293)

### Solution
Quite straightforward: move getting device parameters to a background thread. We might also improve it further in the future: revisit if we need to really collect these data for each event and maybe query it in intervals. [https://github.com/Automattic/Automattic-Tracks-Android/pull/214](https://github.com/Automattic/Automattic-Tracks-Android/pull/214)

## Sentry Profiling
Sentry has [profiling feature](https://docs.sentry.io/platforms/android/profiling/) which works pretty well. The best value, comparing to all other tools, is that it can show **aggregated** data, which is great for identifying performance bottlenecks (e.g. "show me 3 slowest methods (P75) in the whole app).



