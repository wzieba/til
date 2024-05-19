# `PasswordSafe`

IntelliJ Platform has a very convenient API for storing user secrets. Using it as simple as 

```kotlin
// Credentials
fun createCredentialAttributes(): CredentialAttributes {
    return CredentialAttributes(
        generateServiceName("automattic-encrypted-logs", "usernamepassword")
    )
}

// Setting
PasswordSafe.instance.set(
	createCredentialAttributes(),
	Credentials(username, password)
)

// Getting
val usernamePassword = PasswordSafe.instance.get(createCredentialAttributes())
```

we can make sure, that credentials are secure, as underneath it uses platform-specific safe storage APIs. For macOS, it'll be Keychain using Security Framework.

I used it when developing [EncryptedLoggingIntelliJ](https://github.com/Automattic/EncryptedLoggingIntelliJ/) plugin, and I was positively surprised by such convenient API for this use case.
