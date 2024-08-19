# Sharding instrumentation tests

Sharding is a process of **splitting tests into subsets** and **execute them at the same time** on different devices. The first time I've heard this term was in [Cyberpunk 2077](https://cyberpunk.fandom.com/wiki/Cyberpunk_2077_Shards), but that's actually a little different.

When I read about this idea, it sounded appealing, because tests should be either way isolated, so just running them in subsets shouldn't cause any issues (especially if run on *different devices*).

### How?
In Automattic, we use Firebase Test Lab to run instrumentation tests, so [Flank](https://github.com/Flank/flank) and a Gradle Plugin that wraps it, [Fladle](https://github.com/runningcode/fladle/), happened to be a good fit for us.

The idea behind Flank is that after each execution of tests, Flank gathers and updates information about duration of each test. The results are saved in a Cloud bucket.

Then, before executing tests next time, Flank fetches this data and based on it, forms subsets of tests, that the compound duration is projected to be a given time (for us, it was 120s).

Our config is available here: https://github.com/woocommerce/woocommerce-android/blob/34c51ae8cc7188c08a40a6be5c7cbffc322227a3/WooCommerce/build.gradle#L19-L33

### Impact
In case of WooCommerce Android project, we **saved from 5m 36s up to 8m 6s** on each CI pipeline check, which projects to **57h 1m to 82h 29m** per month. This is an outstanding result, taking into account low effort of integration.

![image](https://github.com/user-attachments/assets/6d0bc20f-58b8-48cc-93cd-87fc675b6d5e)

### Costs
Our estimated cost is projected to be extra $17. This makes it a no-brainer for ROI on this.

----

I wish I TILed this earlier!