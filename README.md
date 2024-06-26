grm
====

### What is it?
`grm` is an experimental package manager for GitHub Releases. It is probably only good for installing packages which are distributed as binaries (for example the ones written in Go, Rust). The following popular packages can be installed with `grm`:
 - mozilla/geckodriver
 - gohugoio/hugo
 - go-acme/lego
 - zyedidia/micro
 - ...

`grm` inspects release assets for a binary file and when the file is found, downloads and installs it to `$HOME/.local/bin/` directory.

### How to use it?
```bash
$ ./grm
A package manager for GitHub releases

Usage:
  grm [command]

Available Commands:
  aliases     Print table of known package aliases
  help        Help about any command
  info        Show information about a package
  install     Install a package from GitHub releases
  list        List installed packages
  lock        Lock a package
  release     Create a release in GitHub
  remove      Remove a package
  set         Modify settings
  settings    Print settings
  unlock      Unlock a package
  update      Update installed packages
  version     Print version

Flags:
  -h, --help           help for grm
      --token string   GitHub API token
  -v, --verbose        Enable verbose logging
  -y, --yes            Confirm all

Use "grm [command] --help" for more information about a command.
```

#### How to install specific version of hugo?
```grm
grm install gohugoio/hugo==v0.63.0 -f Linux-64
```

### How to install it?
> Make sure you have `curl` installed (`sudo apt install curl`)
```bash
BINARY=grm_<os (linux|darwin)>_<architecture (amd64|arm64)>.tar.gz
curl -s https://api.github.com/repos/patrick-5546/grm/releases/latest \
  | awk -F'"' '/"browser_download_url":/{print $4}' | grep $BINARY \
  | xargs curl -LOs && tar -xf $BINARY && rm $BINARY && mv grm $HOME/.local/bin/
```
