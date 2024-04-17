# `git filter-repo` 

`filter-repo` is a handy tool that can be used to extract parts of projects to a separate repository **keeping the complete git history of files**.

The idea here is to *filter out* all unnecessary files, leaving only those commits, which affected the files that stayed in the repository. For this reason, it's best to perform this operation on a clean clone, alternatively you can run `git -fdx`.

## Example usages

### `WordPress-MediaPicker-Android`

[https://github.com/wordpress-mobile/WordPress-MediaPicker-Android/pull/14](https://github.com/wordpress-mobile/WordPress-MediaPicker-Android/pull/14)

```shell
git filter-repo --path WordPress/src/main/java/org/wordpress/android/ui/mediapicker/ \  
 --path WordPress/src/test/java/org/wordpress/android/ui/mediapicker/ \  
 --path-regex layout\/media_picker \  
 --path-rename WordPress/src/main/java/org/wordpress/android/ui/:MediaPicker/src/main/java/org/wordpress/android/ \  
 --path-rename WordPress/src/test/java/org/wordpress/android/ui/:MediaPicker/src/test/java/org/wordpress/ \  
 --path-rename WordPress/src/main/res/layout/:MediaPicker/src/main/res/layout/ \ 
 --replace-text ~/expressions.txt


expressions.txt:
org.wordpress.android.ui.mediapicker==>org.wordpress.android.mediapicker
```

The above command was used to extract `mediapicker` packages, with all needed layout files and tests. During the filtering 2 modifications were performed:
- `--path-rename` changed the directories/packages of the files, making it more suitable for the package structure in the new repository
- `--replace-text` which affected `package` and `import` of Java and Kotlin files

Then, we can merge what was left from the original repository, to the new one using `--allow-unrelated-histories` option. It can look like

```shell
git remote add wp ~/WordPress-Android-Temp # directory with filtered repository
git fetch wp
git merge wp/develop --allow-unrelated-histories
git remote remove wp
```

### `Encrypted-Logging`

[https://github.com/Automattic/EncryptedLogging/pull/2](https://github.com/Automattic/EncryptedLogging/pull/2)
[https://github.com/Automattic/EncryptedLogging/pull/3](https://github.com/Automattic/EncryptedLogging/pull/3)

```shell
git filter-repo \
 --path example/src/test/java/org/wordpress/android/fluxc/encryptedlog/EncryptedLogSqlUtilsTest.kt \
 --path-rename example/src/test/java/org/wordpress/android/fluxc/encryptedlog:encryptedlogging/src/test/kotlin/com/automattic/encryptedlogging/encryptedlog \
 --path example/src/androidTest/java/org/wordpress/android/fluxc/release/ReleaseStack_EncryptedLogTest.kt \
 --path-rename example/src/androidTest/java/org/wordpress/android/fluxc/release:encryptedlogging/src/androidTest/kotlin/com/automattic/encryptedlogging/release \
 --path example/src/androidTest/java/org/wordpress/android/fluxc/LogEncrypterTest.kt \
 --path-rename example/src/androidTest/java/org/wordpress/android/fluxc/:encryptedlogging/src/androidTest/kotlin/com/automattic/encryptedlogging/ \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/model/encryptedlogging/EncryptedLog.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/action/EncryptedLogAction.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/model/encryptedlogging/LogEncrypter.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/network/BaseRequest.java \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/network/EncryptedLogUploadRequest.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/network/rest/wpcom/encryptedlog/EncryptedLogRestClient.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/persistence/EncryptedLogSqlUtils.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/store/EncryptedLogStore.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/model/encryptedlogging/EncryptedSecretStreamKey.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/model/encryptedlogging/SecretStreamKey.kt \
 --path fluxc/src/main/java/org/wordpress/android/fluxc/model/encryptedlogging/EncryptionUtils.kt \
 --path-rename fluxc/src/main/java/org/wordpress/android/fluxc/:encryptedlogging/src/main/kotlin/com/automattic/encryptedlogging/ \
 --replace-text ~/expressions.txt


expressions.txt:
org.wordpress.android.fluxc==>com.automattic.encryptedlogging
```

A fairly similar case, but this time files were more scattered around the codebase, so there are more `--path` arguments. What's interesting here is that the migration doesn't have to happen in one merge (see PRs #2 and #3). One can add some files in one merge, then realize that not all needed files were extracted, and add more in another merge.