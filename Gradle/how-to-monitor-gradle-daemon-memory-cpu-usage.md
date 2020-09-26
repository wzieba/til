# How to monitor Gradle daemon CPU & RAM usage (or any other JVM proccess)

1. Install and open VisualVM
```
brew casks install visualvm
```
2. Stop existing daemons
```
./gradlew --stop
```
3. When e.g. running Android compilation, Gradle will spawn two daemons: Gradle Daemon and Kotlin Compile Daemon. To merge those two into one daemon, specify kotlin compiler execution strategy like this:
```
-Dkotlin.compiler.execution.strategy="in-process"
```
4. See results on the right, it should be simillar to this:
<img width="1389" alt="spire_without_desugar" src="https://user-images.githubusercontent.com/5845095/94260063-ca409f80-ff2f-11ea-9a49-2a87f4bac42a.png">

