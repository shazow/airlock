# airlock

Cross environment trust boundaries, safely.

**Status**: README-Driven-Development phase.

More braindumps on the idea here: https://github.com/shazow/shazow.net/issues/88 https://github.com/shazow/shazow.net/issues/87

## Premise

Sandboxes are great, but giving them write access to the host disk is dangerous. What if the sandbox rewrites your git history and injects malware into the dependencies, or your unit tests, or your continuous integration, or something else for next time you run the code outside of the sandbox?

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
- [ ] `airlock receive --path=./bundle.airlock --review=skip --directory=$HOME` (receiving via local path is compatible with readonly shared volumes and out-of-band transports)
- [ ] `airlock review --path=./bundle.airlock`
- [ ] `airlock bundle | airlock send $MAGIC_CODE` powered by magic-wormhole-like discovery transport (built on libp2p?)
- [ ] `airlock receive $MAGIC_CODE`
- [ ] `airlock review` with native review TUI for reviewing git patchsets, git bundles, markdown, config payloads, and other state.
- [ ] `airlock.toml`: Configure bundle contents and bundling mode (e.g. use `git bundle` for git repos, specify branch somehow)
- [ ] `airlock.toml`: Configure review tools per bundle item (e.g. `repo.bundle` should do something like [agentspace-import](https://github.com/shazow/agentspace/blob/34474e130f4efaae1d9d4be5fd8391ba4cdf0ea3/scripts/agentspace-import)).
- [ ] `airlock review` option to promote git changes to a Github pull request (ideally forge-agnostic interface for supporting other forges later).
- [ ] `airlock review` option to delegate to an automated review process (such as a coding agent, or a different kind of program)

## Airlock Bundle Convention

Airlock is composed of an airlock bundle convention and tooling for dealing with it. We can imagine a variety of tools for transporting and reviewing bundles, so let's focus on the bundle convention that we can all converge around.

We try to keep it as simple as possible so that it's easy to do steps manually and to write custom tools that improve our workflows.

- An airlock bundle is a tarball. It can be compressed or not.
- All the paths are assumed to be relative to `$HOME`. The recipient can choose to extract under a different directory.
- Items inside the tarball are named to make it easy to target paths with glob/regexp to map to appropriate review tools. For example: "Review `*.md` with `bat`" or "Review `*.patch` with `$EDITOR`", this will need to become its own convention over time.
- Top-level `README.md` will be reviewed/displayed first, it can include useful context for the reviewer.
- Items inside the bundle can be accompanies with a corresponding `*.md` file to preface the corresponding review with additional context. For example: `my-feature-branch.patch` can be accompanied with `my-feature-branch.patch.md` to act as a coverletter for the patch.
- Expect configs to be included in the bundle, such as `.codex/config.toml` or `.config/git/config` or `.config/airlock/config.toml`.
- Expect assets like logs and images.

We don't need special tools to do this, we can use `tar` and `scp` and `cat` to create bundles, transport them, and review them. The goal of `airlock` is to streamline this workflow, to make it more robust and powerful.

It's important to keep the fundamentals simple so that we can always fall back to them if we need, or build our own better tools.

## Airlock Review

Airlock will come with a native review TUI, here's a sketch of what it may look like:

`airlock receive --path=./bundle.airlock --directory=~/projects/airlock --review=native`

```
README.md:
This is a coverletter that was included in the bundle.

It might be very long, but we'll truncate it after the first few lines
... [Read more]

Receive?
- [ ] `README.md`
- [ Review | Drop | Pick ] `my-feature-branch.git`  + `.md`
- [~] `output.log`
```

We can also decouple the review process into an output similar to `git rebase -i` where each line decides if we pick the entry or not:

`airlock review --path=./bundle.airlock --mode=editor`

```
# README.md:
# This is a coverletter that was included in the bundle.
#
# It might be very long, but we'll truncate it after the first few lines
# ... [truncated 2,810 bytes]
drop README.md                   # 2.8KB, 812 words
pick my-feature-branch.patch     # 6.4KB, +1432 -98 lines, 14 files changes
drop my-feature-branch.patch.md  # 1.1KB, 294 words
drop output.log                  # 988 bytes, 89 lines

# Bundle review: ./bundle.airlock
# Comments are ignored.
# d, drop <path> = skip path from bundle
# p, pick <path> = receive path
```

Any tool can produce a review like this, then it can be piped into `airlock receive --path=./bundle.airlock --directory=~/projects/airlock --review=stdin < bundle.review`

Or more simply: `echo "pick my-feature-branch.patch" | airlock receive --path=./bundle.airlock --directory=~/projects/airlock --review=stdin`

TODO:
- [ ] Include a way to specify additional review inside the editor mode, similar to `exec` mode.


## License

MIT
