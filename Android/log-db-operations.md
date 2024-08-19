# Log SQL statements

It's possible to log all database operations that are performed by an Android application by using `adb`.

To do this, run an emulator with `ROOT` access ([AOSP image, without Google Services](https://developer.android.com/studio/run/managing-avds#system-image)), and then

```sh
adb shell stop
adb shell start
adb root
adb shell setprop log.tag.SQLiteStatements VERBOSE # Controls the printing of SQL statements as they are executed.
adb shell setprop log.tag.SQLiteLog VERBOSE # Controls the printing of informational SQL log messages.
adb shell setprop log.tag.SQLiteTime VERBOSE # Controls the printing of wall-clock time taken to execute SQL statements as they are executed.
```

This works regardless of used ORM. I find it very helpful, especially when I recently worked on a race condition issue in one of our libraries that uses [wellsql](https://github.com/yarolegovich/wellsql) framework, which occasionally doesn't handle concurency in the best way possible.

Source:
- [http://androidxref.com/4.2.2_r1/xref/frameworks/base/core/java/android/database/sqlite/SQLiteDebug.java](http://androidxref.com/4.2.2_r1/xref/frameworks/base/core/java/android/database/sqlite/SQLiteDebug.java)
- [https://stackoverflow.com/a/54963655](https://stackoverflow.com/a/54963655)