# Migrating to KSP

## How kapt and ksp differ?

### Incremental processing
When writing content to a file during processing with ksp, it is necessary to define output mode.

- **Aggregating** means that almost any change in the codebase will consider the input as dirty, (meaning, that the processor needs to run again).
- **Isolating** means that only changes to the specified files will make input dirty.

We also can specify list of source files. It's especially important in isolating mode (change in source files makes input dirty). But why specify source files in aggregating mode, if any change in codebase makes input dirty?

> in the case of `aggregating = true` there is still an optimization available to you if you are careful about originating files. The optimization is for the case where files that are not registered inputs are deleted

Details: https://kotlinlang.slack.com/archives/C013BA8EQSE/p1708451414104879

### Multiple rounds
ksp executes multiple rounds of processing to make sure all symbols it visits are validated. It’s a useful mechanism if our annotation processing depends on other annotation processors.

PR that migrated WordPress-Android internal processor from kapt to ksp: https://github.com/wordpress-mobile/WordPress-Android/pull/20217

# Resources

- ([documentation](https://kotlinlang.org/docs/ksp-overview.html)) Kotlin Symbol Processing API
- ([video](https://href.li/?https://www.youtube.com/watch?v=MMuc5gkKB9s)) Codegen with KSP: A Farewell to Stubs
- ([blog post](https://kt.academy/article/ak-ksp)) Kotlin Symbol Processing
