# JiT and AoT compilations in JVM

## Introduction
The process we usually refer to as "compilation", in a fact is a process of two, consecutive compilations:
1. From `code` (high-level language, Kotlin/Java/Scala, etc.) to the `bytecode`
2. From `bytecode` to `machine code`

When referring to Just in Time (JiT) and Ahead of Time (AoT) compilations, it's always the second type of compilation: from `bytecode` to `machine code`

## `Bytcode` to `machine code`

### Just in Time compilation

During compilation, `bytecode` is read by JVM's `interpreter` which is a module that takes bytecode instruction and assigns corresponding **machine instruction** that is then sent to CPU.

`Interpreter` **is dependant on the architecture of the system**, on which the app is meant to run, as machine instructions vary between different architectures which is a core element behind Java's `WORA` principle (Write Once, Run Anywhere). The downside of the `interpreter` is that it's much slower than direct machine code execution. To diminish this issue, JVM's compilers perform several optimizations.

#### Compiler optimizations
JVM counts how many times the given `bytecode` line is interpreted. If it reaches a given threshold (by default it's 10 000, set by `-XX:CompileThreshold=10000`) it will start compilation (not *just* interpretation) using its `compilers`.  Those methods are called "hot spots".

Optimized machine code will be stored in `code cache` (in Java 8 default max size is 240 MB, `-XX:ReservedCodeCacheSize=240m`). In other words:
> If some code is executed frequently, compile (optimize) it and put it to cache so there's no need to compile it again.

##### Compiler types

JVM has to offer two compilers:

| | Client (C1) | Server (C2) |
| --- | --- | --- |
| Optimization speed | Faster | Slower |
| Optimization efficiency | Worse | Better |
| Resource usage | Smaller | Bigger
| Purpose | GUI apps | Server-side

Before `Java 7` one of the compilers could be chosen. `Java 7` allowed to choose both of them and `Java 8` has this as a default. Using both compilers is called `tiered compilation`.

#### Tiered compilation
In the runtime, stats for all methods execution are registered. If some code is used more frequently, it'll be optimized more heavily. Levels of compilation are as follows:

0. Interpreted code
1. Simple C1 compiled code
2. Limited C1 compiled code
3. Full C1 compiled code
4. C2 compiled code


<img width="1111" alt="Screenshot 2021-01-10 at 19 17 28" src="https://user-images.githubusercontent.com/5845095/104131731-92435800-5378-11eb-865d-e96bdd67e30d.png">

Source: https://youtu.be/sJVenujWGjs?t=261


### Ahead of Time compilation
To save resources and time during program runtime, we could perform those compilations before and just use ready machine code right?

The answer is: yes, it's possible but as interpretation to `machine code` **requires knowledge about system architecture** (e.g. x86, ARM), our JVM program will stop working on those platforms that are not compatible with the one that AoT compilation was based for.

OpenJDK offers partial (in form of `*.so` library) AoT with the tool called [`jaotc`](https://docs.oracle.com/en/java/javase/13/docs/specs/man/jaotc.html):

<img width="1596" alt="Screenshot 2021-01-10 at 19 22 22" src="https://user-images.githubusercontent.com/5845095/104131810-2f9e8c00-5379-11eb-8125-f6e79a2ab0c6.png">

Source: https://youtu.be/sJVenujWGjs?t=519

AoT compiled code is moved directly to `code cache` which results with ready-to-use, compiled and optimized `machine code`:

<img width="1566" alt="Screenshot 2021-01-10 at 19 24 55" src="https://user-images.githubusercontent.com/5845095/104131858-8efc9c00-5379-11eb-82d0-f437f6a74ef4.png">

Source: https://youtu.be/sJVenujWGjs?t=529

[`GraalVM's Native Image`](https://www.graalvm.org/reference-manual/native-image/) is one of the tools that offers fully AoT compiled JVM programs. This of course comes with great performance improvements and a lack of ability to run the same code on many architectures.

## Sources:
- (eng) https://www.oreilly.com/library/view/java-performance-2nd/9781492056102/ch04.html
- (eng) https://www.oracle.com/technical-resources/articles/java/architect-evans-pt1.html
- (eng) https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html
- (eng) https://youtu.be/sJVenujWGjs
- (pl) https://bottega.com.pl/pdf/materialy/jvm/jvm1.pdf
