# ATM instrumentation tests do not work with core desugaring

More details can be found on [issue tracker](https://issuetracker.google.com/issues/158018485) or in [Medium article](https://medium.com/androiddevelopers/support-for-newer-java-language-apis-bca79fc8ef65)

Long story short: at this moment (22/09/2020) it is not possible to run instrumentation tests while using desugaring and obfuscation. Tests simply won't start.

