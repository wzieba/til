# Migrating to KSP

## How kapt and ksp differ?

### Incremental processing
When writing content to a file during processing with ksp, it is necessary to define output mode.

**Aggregating** means that almost any change in the codebase will consider the input as dirty, (meaning, that the processor needs to run again).
**Isolating** means that only changes to the specified files will make input dirty.

### Multiple rounds
ksp executes multiple rounds of processing to make sure all symbols it visits are validated. It’s a useful mechanism if our annotation processing depends on other annotation processors.

PR that migrated WordPress-Android internal processor from kapt to ksp: https://github.com/wordpress-mobile/WordPress-Android/pull/20217
