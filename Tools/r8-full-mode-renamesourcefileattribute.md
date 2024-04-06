# R8 full mode enables `-renamesourcefileattribute` by default

Recently, I've spotted, that stack traces of issues reported to Sentry from Pocket Casts Android are linked to `SourceFile` instead of the real file name.

![image](https://github.com/getsentry/rust-proguard/assets/5845095/9427db0c-cab7-41a0-ac98-66d4715a9671)

Usually, this happens when `-renamesourcefileattribute` is used:

> `-renamesourcefileattribute`
	Specifies a constant string to be put in the `SourceFile` attributes (and `SourceDir` attributes) of the class files.

I made sure, though, that the project does not use this option, also in R8 generated configuration, which combines all R8 configurations during the build.

Later, I realized that R8 **whenever project uses full mode, `-renamesourcefileattribute` is also enabled** (even if not mentioned in the configuration)! Here's the reproduction: [wzieba/R8FullModeRenamesSources](https://github.com/wzieba/R8FullModeRenamesSources).

This motivated me to ask a question at Android Study Group ([Slack](https://androidstudygroup.slack.com/archives/C6MKCJR8V/p1711987719916089)), which confirmed that this behavior is intended ([R8 source code](https://r8.googlesource.com/r8/+/f0816ebb923855c8946ed8008629e6a73f345ad3/src/main/java/com/android/tools/r8/naming/SourceFileRewriter.java#23)) and mentioning it was missed from R8 full mode documentation. Thanks to this discussion, R8 docs are now [updated](https://r8-review.googlesource.com/c/r8/+/90721) with: 

>## R8 full mode
>...
> - When optimizing or minifying the `SourceFile` attribute will always be
rewritten to `SourceFile` unless `-renamesourcefileattribute` is used in which
case the provided value is used. The original source file name is in the mapping
file and when optimizing or minifying a mapping file is always produced.

In other words: `SourceFile` is always rewritten (name of the file will be obfuscated), but if e.g. `-renamesourcefileattribute FooBar` is used, then istead of `SourceFile` string, the stack trace will point to `FooBar`.

# Sentry `-renamesourcefileattribute` support

As I knew, the reason I see `SourceFile` in Sentry dashboard, I still didn't know why it's not deobfuscated. This leads me to [getsentry/rust-proguard](https://github.com/getsentry/rust-proguard)project - Sentry's R8/Proguard rust-based custom deobfuscator.

Upon browsing the repository, I've realized that the project didn't support recovering file names (`SourceFile` -> real file name) which was described in [this issue](https://github.com/getsentry/rust-proguard/issues/29).

I decided to make the contribution, resulting in this PR: [Support symbolicated file names](https://github.com/getsentry/rust-proguard/pull/36). The Sentry team was very welcoming, and we together worked towards merging the PR. In the end, the feature of supporting deobfuscating file names should be soon available, unblocking many Sentry features for projects that use `-renamesourcefileattribute` or R8 full mode 😊.