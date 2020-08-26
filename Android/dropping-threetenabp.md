# Dropping ThreeTenABP in favor of Core Library Desugaring

If one uses AGP version 4.0+, it is possible to drop support for commonly 
used library from @JakeWharton which backports `JSR-310 Date and Time API` for
Android. Friendly reminder here: pre-AGP 4.0+ projects with `minSdk` below `26` can't natively
use any Java 8 APIs.

### Enable desugaring
Two things must be done:
1. enable `codeLibraryDesugaringEnabled` in `android.compileOptions`
1. add desugar lib by `coreLibraryDesugaring` configuration

```groovy
android {
...
 compileOptions {
    coreLibraryDesugaringEnabled true
 }
}

dependencies {
  coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.0.10'
}
```

## The issue of timezone databases

On pre-29 API devices, timezone data is provided by the system itself,
so its validity is dependant on OEMs and users, which my decide not to
update Android to the newest version (or just can't).

With ThreeTenABP it was not a problem as ThreeTenABP provides timezone
information with itself as a standard Android asset.

With Java 8 it is a problem as java.time takes system-level timezone
information. To provide timezone data seperately from Android system while
using desugaring, one might use @ZacSweers [ticktock](https://github.com/ZacSweers/ticktock).
This library allows to **inlude own timezone data during desugaring**. For details of the process go
to blogpost: https://www.zacsweers.dev/ticktock-desugaring-timezones/

