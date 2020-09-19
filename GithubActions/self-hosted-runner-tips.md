# Self-hosted runners tips and tricks

Recently I've was setting up self-hosted Github Actions runner. I aimed to make them independent as much as I could, so I've decided to use `Docker` for that. Here are some of my tip&tricks/gotchas I've encountered

## Fallback mechanism when `Git` version < 2.18

If your self-hosted machine has installed `Git` with version lower than 2.18, the `actions/checkout@v2` will fallback to REST API. It is explicitly written in [docs](https://github.com/actions/checkout#whats-new) but quite suprising.
What does it mean for the project? E.g. incorrectly computed version name, if you use `Git`-based versioning strategy.

## You can't connect usb device to dockerized linux, when hosting on mac

In this scenario we've got macOS machine (Mac mini, you guessed it) and for each worker we create seperate, self-sufficient `Docker` container.

Now, we would like to run UI tests on real device in one of those `Docker` containers, but we can't as it's [impossible to pass usb device to the container if macOS is host](https://docs.docker.com/docker-for-mac/faqs/#can-i-pass-through-a-usb-device-to-a-container)

## Dockerized linux on mac is really slow (and has memory issues)

Builds that usually would run ~10 minutes on Github Actions machines, did run ~30 minutes on dockerized linux hosted by macOS.
I've learned, that `Docker` on macOS doesn't run on the machine itself but rather on some really small, unix based VM (is it HyperKit?) so it'll always be quite slow.
There's several resources providing help in optimizing `Docker` on macOS but my limitted resources didn't allow to explore it further.

## Self-hosted runners are not idempotent

This one is the most imporant for me. In my opinion, CI system should be completaly self-sufficient and idempotent. This is how Github Actions work in not self-hosted envirnoment: every time we have to e.g. set Java or Ruby version, we have to fetch depedndencies etc. In self-hosted runners we always run build against the same machine. If it's Docker container it will run in the container, but it will always be the same container, with the same set of settings and changes that previous builds might have done.

## Github Actions does not provide Docker images of its machines

In contrast to [Bitrise](https://github.com/bitrise-io/android) or [CircleCI](https://github.com/CircleCI-Public/example-images/tree/master/android), Github Actions doesn't provide `Docker` images of the environment.
They provide it in some different way here: https://github.com/actions/virtual-environments but I didn't know where to start with this in the first place.

---

This experience was pain. I dropped the idea of running and managing self-hosted runners. I hoped it'll be much, much easier. Maybe next time.

