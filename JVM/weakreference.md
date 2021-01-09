# WeakReference

**Weak Reference**
> A **weak reference** is a reference made that is not strong enough to make the object to remain in memory. So, weak references can let the garbage collector decide an object’s reachability and whether the object in question should be kept in memory or not.
>
> Weak references need to be declared explicitly as by default Java marks a reference as a strong reference.

~ https://javatutorial.net/weak-references-in-java

**Reference Queue**
> (...) **reference queue** is a way for the GC to inform the program that a certain object is no longer reachable.

> If the garbage collector discovers an object that is weakly reachable, the following occurs:
>
>1.  The **WeakReference** object's referent field is set to null, thereby making it not refer to the heap object any longer.
>2.  The heap object that had been referenced by the **WeakReference** is declared finalizable.
>3.  The **WeakReference object is added to its ReferenceQueue**. Then the heap object's finalize() method is run and its memory freed.


~ http://learningviacode.blogspot.com/2014/02/reference-queues.html

### Playground

**Source code**
```kotlin
class Foo

fun main() {

    var foo: Foo? = Foo()
    val queue = ReferenceQueue<Foo?>()
    val weakReference = WeakReference(foo, queue)

    fun print() {
        println("referant:\t\t${weakReference.get()}\nis enqueued:\t${weakReference.isEnqueued}\nqueue empty:\t${queue.poll() == null}")
    }

    fun gc() {
        println("--- GC ---")
        System.gc()
    }

    print()

    gc()
    print()

    foo = null
    Thread.sleep(1000)
    gc()
    print()
}
```
**Output**
```
referant:	Foo@28d93b30
is enqueued:	false
queue empty:	true
--- GC ---
referant:	Foo@28d93b30
is enqueued:	false
queue empty:	true
--- GC ---
referant:	null
is enqueued:	true
queue empty:	false
```

**What's happening here?**
1. First output is a safe check: we've got object stored in `WeakReference`, which in context of `Reference` is called **`referant`**, it's not enqueued for garbage collection and the `ReferenceQueue` is empty.
2. In the second block Garbage Collector is instructed to run but as we have **strong** reference to the `foo` object, running it doesn't change anything.
3. We loose strong reference to the `foo` so Garbage Collector can start its work. As presented, the refarnt has been collected, the reference has been enqueued to collection and the `ReferenceQueue` is not empty anymore

**Why Thread.sleep(1000)?**
> (...) even if  `WeakReference.get()`  returns  `null`  it is not guarantee that  `WeakReference.enqueue`  will be  `true`  or  `ReferenceQueue.poll`  wont return  `null`

~ https://stackoverflow.com/a/48432416

Without giving some time for GC, we can see non deterministic results of setting reference as enqueued and putting it to the reference queue.

### Is it possible to get back collected object with `ReferenceQueue`?

> (...) with the ReferenceQ there is no way to reach the released java object

~ http://learningviacode.blogspot.com/2014/02/reference-queues.html

👆After setting `null` value on weakly referenced object and garbage collecting, we can poll the `WeakReference` object from `ReferenceQueue`. Although it's the same `WeakReference` object, it still points to `null` though we can't access it and re-initialize collected object:

**Source code**
```kotlin
class Foo

fun main() {
    var foo: Foo? = Foo()
    val queue = ReferenceQueue<Foo?>()
    val weakReference = WeakReference(foo, queue)

    foo = null
  System.gc()
    println("WeakReference points to:\t\t\t\t${weakReference.get()}")
    val lastItem = queue.poll()
    println("Last item on queue the same reference?:\t${weakReference == lastItem}")
    println("Does it point to original object?:\t\t${lastItem.get() != null}")
}
```

**Output**
```
WeakReference points to:                null
Last item on queue the same reference?:	true
Does it point to original object?:      false
```

