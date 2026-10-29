# patrick-5546/grm

This repository is a fork of [jsnjack/grm](https://github.com/jsnjack/grm). It
follows upstream releases and adds these commits on top of each one:

- A `bin_dir` setting. `grm set bin_dir <absolute path>` installs binaries to
  another directory, and binary files use sudo only when the current user
  cannot write to that directory. This change is proposed upstream in
  [jsnjack/grm#20](https://github.com/jsnjack/grm/pull/20).
- The `grm` alias and `make release` point at this fork.
- This file and `.github/workflows/release.yml`.

A release tag here uses the upstream version it is based on. For example,
`v2.0.8` in this fork is upstream `v2.0.8` plus the commits above.

## Install

```bash
BINARY=grm_<os (linux|darwin)>_<architecture (amd64|arm64)>.tar.gz
BIN_DIR=$HOME/.local/bin
curl -s https://api.github.com/repos/patrick-5546/grm/releases/latest \
  | awk -F'"' '/"browser_download_url":/{print $4}' | grep $BINARY \
  | xargs curl -LOs && tar -xf $BINARY && rm $BINARY && mv grm $BIN_DIR/
$BIN_DIR/grm set bin_dir $BIN_DIR
```

## Sync with upstream

Set up a clone once. Upstream tags are fetched into `refs/upstream-tags/` so
they do not clash with this fork's tags of the same name.

```bash
git clone https://github.com/patrick-5546/grm.git && cd grm
git remote add upstream https://github.com/jsnjack/grm.git
git config remote.upstream.tagOpt --no-tags
```

For each upstream release, rebase the fork commits onto its tag, push, and tag
the result with the same version:

```bash
TAG=v2.0.9
git fetch upstream '+refs/tags/*:refs/upstream-tags/*'
git switch master && git pull --ff-only
git rebase "refs/upstream-tags/$TAG"
docker run --rm -v "$PWD:/src:ro" -w /src fedora:latest \
  sh -c 'dnf install -y -q golang >/dev/null && go test ./...'
git push --force-with-lease origin master
git tag "$TAG" && git push origin "$TAG"
```

Upstream's tests expect a Fedora host, where `rpm` is installed and `dpkg` is
not, so they run in a Fedora container. Pushing the tag runs
`.github/workflows/release.yml`, which runs the tests on Fedora, builds with the
tag as the version, and publishes `grm` and
`grm_{linux,darwin}_{amd64,arm64}.tar.gz` as the latest release.

Fork commit messages must not start with a version prefix such as `m ` or
`:p:`, so that [monova](https://github.com/jsnjack/monova) still computes the
upstream version.
If upstream merges the `bin_dir` change, the rebase skips the fork's copy when
the patches match; otherwise remove it with `git rebase -i`.
