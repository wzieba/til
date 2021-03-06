# Karabiner Elements config

## Update 15.09.2020

Goku supports only `Complex modifications` for Karabiner. `Complex modifications` work pretty bad, specially
for single-key mappers. E.g. after `left cmd` -> `left option` and `left option` -> `left cmd` modification,
pressing (in following order) `left option + left ctrl` gave `left ctrl + left option` combination, but 
pressing `left ctrl + left option` was logging... `left command + left option` which made no sense.

After strugling for some time with finding any decent solution, I gave up on that and created `simple modifications`
in Karabiner GUI.

## Old post

Karabiner Elements is keyboard customizer. Its generated from GUI config file is (`karabiner.json`) terrible:
1. It creates new file every time (so it's impossible to keep it in dotfile repo based on symlinks
1. It's very difficult to read - a lot of boilerplate and hard-to-follow structure

## Goku

There's a better approach provided by [Goku tool](https://github.com/yqrashawn/GokuRakuJoudo). It takes `.edn` and generates `karabiner.json` from it.

## Goku and Dotbot

If you want to integrate Karabiner config generation from `.edn` file you have to:
1. Provide minimal `karabiner.json` config with profile named `Default`. See [this stub][karabiner_stub] from my dotfiles repository.
1. Copy this `karabiner.json` to `~./config/karabiner/karabiner.json`
1. Link `karabiner.edn` to `~./config/karabiner.edn`
1. Run `goku` in command line

Full working example can be found in this commit: [c759d51][c759d51]

[karabiner_stub]: https://github.com/wzieba/dotfiles/blob/c759d5197e1100aed0cbd6aea2a98d0f93651288/config/karabiner/karabiner.json
[c759d51]: https://github.com/wzieba/dotfiles/commit/c759d5197e1100aed0cbd6aea2a98d0f93651288

