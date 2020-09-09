# When Dagger's @Singleton is created twice?

Assumptions: 
1. `DaggerAndroid` is used
1. Main Application `@Component` has `@Module` with `@Subcomponent` in it


When you want to inject `Fragment` dependencies from subcomponent, you have to 
call for Main Application `@Component` in order to fetch exposed `@Subcomponent`.

This can look like this:
```kotlin
override fun onAttach(context: Context) {
    (requireActivity().application as HasFeatureComponent).featureComponent().create().inject(this)
    super.onAttach(context)
}
```

When `Application` setup looks like this:
```kotlin
override fun applicationInjector() = DaggerMainComponent.factory().create(this)

override fun featureComponent() = (applicationInjector() as MainComponent).featureComponent()
```

execution of `featureComponent()` will cause creating `DaggerMainComponent` **every single call**.

This means, that few instances of `DaggerMainComponent` will be created, which
results, among others, with creating `@Singleton` instance for every created component.

### Fix?

We have to create one `DaggerMainComponent` and reuse it later:
```kotlin
lateinit var component: MainComponent

override fun applicationInjector(): AndroidInjector<out DaggerApplication> {
    component = DaggerMainComponent.factory().create(this)
    return component
}

override fun featureComponent(): FeatureComponent.Factory = component.featureComponent()
```


