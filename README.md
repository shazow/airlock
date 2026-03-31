# airlock
Safely cross environment trust boundaries.

**Status**: README-Driven-Development phase.

Some braindumps on the idea here: https://github.com/shazow/shazow.net/issues/88 https://github.com/shazow/shazow.net/issues/87

## Premise

Sandboxes are great, but giving them write access to the host disk is dangerous. What if the sandbox rewrites your git history and injects malware into the dependencies or into your unit tests, for next time you run it?

We should treat development environment as opaque boxes of work. We send some state to work on, it does it in a dangerous way, then we receive some valuable byproduct that we must review before accepting into the host.

Airlock is a protocol and tool for facilitating this workflow.

## Goals:

- A safe way to send, receive, and review git repo state changes (not just patches).
- A program should be able to deliver a bundle anonymously, without a Github account.
- A human or program should be able to review the bundle safely without executing any remote code.
- A bundle should be convertible into a traditional pull request, even non-interactively.
- Not specific to Github. Ultimately should be usable in a fully headless way, bundles flying between machine to machine to machine.

## Features

- [ ] `airlock bundle FILES... > bundle.airlock`
- [ ] `airlock receive --path=./bundle.airlock --review=skip` (receiving via local path is compatible with readonly shared volumes and out-of-band transports)
- [ ] `airlock review --path=./bundle.airlock`
- [ ] `airlock bundle | airlock send $MAGIC_CODE` powered by magic-wormhole-like discovery transport (built on libp2p?)
- [ ] `airlock receive $MAGIC_CODE`
- [ ] `airlock review` with native review TUI for reviewing git patchsets, git bundles, markdown, config payloads, and other state.
- [ ] `airlock.toml`: Configure bundle contents and bundling mode (e.g. use `git bundle` for git repos, specify branch somehow)
- [ ] `airlock.toml`: Configure review tools per bundle item (e.g. `repo.bundle` should do something like [agentspace-import](https://github.com/shazow/agentspace/blob/34474e130f4efaae1d9d4be5fd8391ba4cdf0ea3/scripts/agentspace-import)).
- [ ] `airlock review` option to promote git changes to a Github pull request (ideally forge-agnostic interface for supporting other forges later).

## License

MIT
