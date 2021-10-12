# GOKP Documentation
This is the documentation page for [gokp](https://github.com/christianh814/gokp)

GOKP aims to be a GitOps native Kubernetes Platform. You can read more at the above link. This repo is intended to be the source of documentation for the PoC. The documentation (like the PoC) is "best effort" and is also in a "Proof of Concept" state. Please feel free to submit a PR if you find something is missing.

* [Quickstart AWS](docs/aws-quickstart.md)
* [Quickstart Docker](docs/docker-quickstart.md)

# Getting The Binary

I've hand tested this on Ubuntu 21.04, Fedora 34, and Mac OS X Big
Sur (11.6) all on X86_64.

## Build From Source

```shell
go install github.com/christianh814/gokp
```

If you don't have your `GOBIN` in your `PATH`, you need to set that

```shell
export PATH=$GOBIN:$PATH
```

## Download

Download Binary

```
sudo wget -O /usr/local/bin/gokp https://github.com/christianh814/gokp/releases/download/v.0.0.1/gokp
```

Make sure to make it executable

```
sudo chmod +x /usr/local/bin/gokp
```

## Bash Completion

Now you can get bash completion.

```shell
source <(gokp completion bash)
```

# Questions?

Any questions, I can be found on the [Kubernetes Slack](https://slack.k8s.io/), I am the user `@christianh814`.
