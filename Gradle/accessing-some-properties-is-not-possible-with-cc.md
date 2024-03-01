# Accessing start parameters in Gradle plugin is impossible when using configuration cache

- Gradle forum conversation: https://discuss.gradle.org/t/is-there-a-way-to-get-up-to-date-gradle-start-parameters-with-configuration-cache/47806
- MCVE: https://github.com/wzieba/GetGradleConfigurationInPluginRepro

Currently, it's not possible to get up to date `org.gradle.StartParameter` in a Gradle plugin that uses configuration cache. The reason is that changing starting parameters, like `max-workers` or `configure-on-demand` is not **invalidating configuration cache**, so the parameters won't be updated in the runtime of the plugin.

The exception is `taskNames` as configuration cache is generated per requested tasks, so calling a diffrent set of tasks  will either invalidate cache or cause Gradle to use configuration cache for these task (and return expected `taskNames`).

For this reason I've removed some parameters from Automattic's measure-builds plugin in https://github.com/Automattic/measure-builds-gradle-plugin/pull/35. It's better for us to reduce build times, than to measure not crucial parameters.

This problem exsists in other repositories as well, e.g. https://github.com/cdsap/Talaiot/issues/390
