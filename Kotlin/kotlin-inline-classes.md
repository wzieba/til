# Inline classes in Kotlin

Kotlin language has concept of `inline class`. It's safe to say that it's enhanced `alias` as `inline class` can have only one member:

```kotlin
inline class PostalCode(val value: String)
```

This class will be inlined to plain `String` in runtime.

## `inline class` vs. `alias`

`inline class` is a safer choice in terms of methods' signstures.
Take a look at this snippet:

```kotlin
inline class PostalCode(val value: String)

typealias PostalAlias = String

fun acceptPostalCode(postalCode: PostalCode) {}
fun acceptPostalAlias(postalAlias: PostalAlias) {}

fun main() {

    val foobarCode: String = "123456"

    acceptPostalAlias(foobarCode) // OK. As PostalAlias is String, compiler will accept foobarCode even if it's distinctly String
    acceptPostalCode(foobarCode) // Not OK. The method accepts only members of class PostalCode, foobarCode is not one
}
```

## Recent updates on `inline class`

Kotlin 1.4.30 introduced some improvements. The one that I like is that `init` blocks are available now:

```kotlin
inline class PostalCode(val value: String) {
    init {
        require(value.length == 6)
    }
}
```

That can be nice and usefull addition.

## Java compatibility

It's not possible to call methods that accepts `inline class`es in signature in Java.

### It's not possible to create `inline class` object in Java

The reason is that Kotlin will handle it in generated code as class with `private` constructor with `public static` methods, which will invoke the contructor. The problem is that those classes will be named with `-` character which is not supported in Java. Take a look at sample generated code:

```java
import kotlin.Metadata;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;
import org.jetbrains.annotations.Nullable;

@Metadata(
   mv = {1, 4, 0},
   bv = {1, 0, 3},
   k = 1,
   d1 = {"\u0000\"\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010\u000e\n\u0002\b\u0005\n\u0002\u0010\u000b\n\u0002\b\u0002\n\u0002\u0010\b\n\u0002\b\u0002\b\u0086@\u0018\u00002\u00020\u0001B\u0012\u0012\u0006\u0010\u0002\u001a\u00020\u0003ø\u0001\u0000¢\u0006\u0004\b\u0004\u0010\u0005J\u0013\u0010\b\u001a\u00020\t2\b\u0010\n\u001a\u0004\u0018\u00010\u0001HÖ\u0003J\t\u0010\u000b\u001a\u00020\fHÖ\u0001J\t\u0010\r\u001a\u00020\u0003HÖ\u0001R\u0011\u0010\u0002\u001a\u00020\u0003¢\u0006\b\n\u0000\u001a\u0004\b\u0006\u0010\u0007ø\u0001\u0000\u0082\u0002\u0004\n\u0002\b\u0019¨\u0006\u000e"},
   d2 = {"LPostalCode;", "", "value", "", "constructor-impl", "(Ljava/lang/String;)Ljava/lang/String;", "getValue", "()Ljava/lang/String;", "equals", "", "other", "hashCode", "", "toString", "InlineClassesPlayground"}
)
public final class PostalCode {
   @NotNull
   private final String value;

   @NotNull
   public final String getValue() {
      return this.value;
   }

   // $FF: synthetic method
   private PostalCode(@NotNull String value) {
      Intrinsics.checkNotNullParameter(value, "value");
      super();
      this.value = value;
   }

   @NotNull
   public static String constructor_impl/* $FF was: constructor-impl*/(@NotNull String value) {
      Intrinsics.checkNotNullParameter(value, "value");
      return value;
   }

   // $FF: synthetic method
   @NotNull
   public static final PostalCode box_impl/* $FF was: box-impl*/(@NotNull String v) {
      Intrinsics.checkNotNullParameter(v, "v");
      return new PostalCode(v);
   }

   @NotNull
   public static String toString_impl/* $FF was: toString-impl*/(String var0) {
      return "PostalCode(value=" + var0 + ")";
   }

   public static int hashCode_impl/* $FF was: hashCode-impl*/(String var0) {
      return var0 != null ? var0.hashCode() : 0;
   }

   public static boolean equals_impl/* $FF was: equals-impl*/(String var0, @Nullable Object var1) {
      if (var1 instanceof PostalCode) {
         String var2 = ((PostalCode)var1).unbox-impl();
         if (Intrinsics.areEqual(var0, var2)) {
            return true;
         }
      }

      return false;
   }

   public static final boolean equals_impl0/* $FF was: equals-impl0*/(@NotNull String p1, @NotNull String p2) {
      return Intrinsics.areEqual(p1, p2);
   }

   // $FF: synthetic method
   @NotNull
   public final String unbox_impl/* $FF was: unbox-impl*/() {
      return this.value;
   }

   public String toString() {
      return toString-impl(this.value);
   }

   public int hashCode() {
      return hashCode-impl(this.value);
   }

   public boolean equals(Object var1) {
      return equals-impl(this.value, var1);
   }
}
```

so none of those static methods can be invoked from Java class.

### It's not possible to invoke method wich accepts `inline class` in signature from Java

Assume having Kotlin file:

```kotlin

inline class PostalCode(val value: String)

fun formatPostalCode(postalCode: PostalCode): String {
    return "#${postalCode.value}"
}
```

Because of managling, this method will look like this in Java generated code:
```java
public final class WrapperKt {
   @NotNull
   public static final String formatPostalCode_Egzjoak/* $FF was: formatPostalCode-Egzjoak*/(@NotNull String postalCode) {
      Intrinsics.checkNotNullParameter(postalCode, "postalCode");
      return '#' + postalCode;
   }
}
```

as you can see here again the original name of the method contained `-` symbol which is prohibited in Java methods signatures. This means that this method is not available in Java. More about mangling: https://kotlinlang.org/docs/reference/inline-classes.html#mangling

## More links

- https://kotlinlang.org/docs/reference/inline-classes.html
- https://msfjarvis.dev/posts/improvements-to-inline-classes-in-kotlin-1-4-30
