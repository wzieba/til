# Observable#switchMapCompletable does not complete

> This already got me twice

### When flatMapCompletable won't complete?

```kotlin
Observable.create<String> { emitter ->
    emitter.onNext("foobar")
}.flatMapCompletable {
    Completable.complete()
}.test()
    .assertComplete()
```

This won't pass as `emitter` doesn't call `onComplete`. My intuition was
that `flatMapCompletable` will take first element, map it to `Completable`
and then complete the stream. This intuition is not correct. 

Looking at documentation:

![ObservableFlatMapCompletable](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/flatMapCompletable.o.png)

`flatMapCompletable` will complete only when `Observable` completes.


### When flatMapCompletable will complete?

```kotlin
Observable.create<String> { emitter ->
    emitter.onNext("foobar")
}.firstOrError()
    .flatMapCompletable {
        Completable.complete()
    }.test()
    .assertComplete()
```

This will pass, as we map `Observable` to `Single` with `Observable#firstOrError`.
`Single#flatMapCompletable` "completes based on applying a specified function
to the item emitted by the source `Single`"

![SingleFlatMapCompletable](https://raw.github.com/wiki/ReactiveX/RxJava/images/rx-operators/Single.flatMapCompletable.png)

