![kubedee logo](docs/logo/kubedee.png)

[![builds.sr.ht status](https://builds.sr.ht/~schu/kubedee.svg)](https://builds.sr.ht/~schu/kubedee?)

Fast multi-node Kubernetes (>= 1.19) development and test clusters on [Incus](https://github.com/lxc/incus) or [LXD](https://github.com/canonical/lxd).

Under the hood, [CRI-O](https://github.com/kubernetes-incubator/cri-o) is used
as container runtime and [Flannel](https://github.com/coreos/flannel) for
networking.

For questions or feedback, please open an issue.

## Requirements

* [Incus](https://github.com/lxc/incus)/[LXD](https://github.com/canonical/lxd).
  * Make sure your user is member of the `incus`/`lxd` group (see `incusd --group …` / `lxd --group …`)
  * btrfs is used a storage driver currently and required
* [cfssl](https://github.com/cloudflare/cfssl) with cfssljson
* [jq](https://stedolan.github.io/jq/)
* kubectl

## Installation

kubedee is meant to and easily installed out of git. Clone the repository
and link `kubedee` from a directory in your `$PATH`. Example:

```
cd ~/code
git clone https://github.com/schu/kubedee
cd ~/bin
ln -s ~/code/kubedee/kubedee
```

That's it!

kubedee stores all data in `~/.local/share/kubedee/...`. kubedee LXD resources
have a `kubedee-` prefix.

`KUBEDEE_DEBUG=1` enables verbose debugging output (`set -x`).

## Usage

### Getting started

kubedee can install clusters based on an upstream version of Kubernetes
or your own build.

To install an upstream version, use `--kubernetes-version` to specify
the release (Git tag) that you want to install. For example:

```
kubedee up test --kubernetes-version v1.21.1
```

To install a local build, specify the location of the binaries
(`kube-apiserver` etc.) with `--bin-dir`. For example:

```
kubedee up test --bin-dir /path/to/my/kubernetes/binaries
```

The default for `--bin-dir` is `./_output/bin/` and thus matches the
default location after running `make` in the Kubernetes repository.
So in a typical development workflow `--bin-dir` doesn't need to be
specified.

Note: after the installation or upgrade of kubedee, kubedee requires some
extra time to download and update cached packages and images once.

With a SSD, up-to-date caches and images, setting up a cluster usually takes
less than 60 seconds for a four node cluster (etcd, controller, 2x worker).

```
[...]

Switched to context "kubedee-test".

==> Cluster test started
==> kubectl config current-context set to kubedee-test

==> Cluster nodes can be accessed with 'lxc exec <name> bash'
==> Cluster files can be found in '/home/schu/.local/share/kubedee/clusters/test'

==> Current node status is (should be ready soon):
NAME                         STATUS     ROLES    AGE   VERSION
kubedee-test-controller      NotReady   master   16s   v1.21.1
kubedee-test-worker-2ma3em   NotReady   node     9s    v1.21.1
kubedee-test-worker-zm8ikt   NotReady   node     2s    v1.21.1
```

kubectl's current-context has been changed to the new cluster automatically.

### Incus/LXD VM setup notes

It might be the case that your particular system prevents any user from using
`io_uring`, due to a relatively-recent-at-the-time-of-writing outpour of related
CVEs. This is usually signalled by the `failed to init linux io_uring ring` error.

Should that be the case, in order to spawn VMs through Incus/LXD, the operator
would need to either run the respective daemon with `CAP_SYS_ADMIN` priviledges
or add it's user to a group designated to access `io_uring`. Example follows:

```
IO_URING_GID=666 IO_URING_GNAME="io_uring"
groupadd -r -g "${IO_URING_GID}" "${IO_URING_GNAME}"  # create io_uring group
sysctl -w kernel.io_uring_group="${IO_URING_GID}"  # designate it as such by gid
gpasswd -a incus "${IO_URING_GNAME}"  # add respective daemon's user to said group
systemctl restart incus  # go nuts
```

### Cheatsheet

List the available clusters:

```
kubedee [list]
```

Start a cluster with less/more worker nodes than the default of 2:

```
kubedee up --num-workers 4 <cluster-name>
```

Start a new worker node in an existing cluster:

```
kubedee start-worker <cluster-name>
```

Delete a cluster:

```
kubedee delete <cluster-name>
```

Configure the `kubectl` env:

```
eval $(kubedee kubectl-env <cluster-name>)
```

Configure the `etcdctl` env:

```
eval $(kubedee etcd-env <cluster-name>)
```

See all available commands and options:

```
kubedee help
```

## Smoke test

kubedee has a `smoke-test` subcommand:

```
kubedee smoke-test <cluster-name>
```

[freenode]: https://freenode.net/
