# bean

Creating a boot ISO with containerd/buildkitd, for lima.

![green beans](assets/beans.png)

Lima:

Lima launches Linux virtual machines, using [QEMU](https://www.qemu.org/).

> An image has to satisfy the following requirements:
>
> - systemd or OpenRC
> - cloud-init
> - The following binaries to be preinstalled:
>   - `sudo`
>   - `sshfs`
>   - `newuidmap` and `newgidmap` (shadow)

Format:

- `disk.img`
  - kernel (bzImage)
  - rootfs (rootfs.ext4)
- `cidata.iso`
  - `user-data` (cloud-init)
  - generated by: `limactl`

Bootloader:

- UEFI (instead of BIOS)
- GRUB

Architecture:

- amd64 (`x86_64`)
- arm64 (`aarch64`)

Software:

- `sshd`
- `containerd`
- `buildkitd`
- `nerdctl`

Docker:

`nerdctl` is a Docker-compatible CLI for contai**nerd**.  🤓

nerdctl is a non-core sub-project of CNCF's [containerd](https://containerd.io/).

```console
# nerdctl build -t foo .
# nerdctl run -it --rm foo
```

Toolbox:

The regular image does **not** have a package manager,
but it is possible to install a toolbox container.

The default `toolbox` uses the `alpine` distribution.
That makes it possible to install `apk` packages.

`2.7M	/var/lib/toolbox/alpine_latest.tar`

Based on [CoreOS Toolbox](https://github.com/coreos/toolbox), but using nerdctl.

Design:

This OS follows a layered "three images" approach:

1) Base OS, based on musl/busybox (like [Alpine](https://alpinelinux.org/))

2) "boot2nerdctl", adds containerd and buildkitd

3) Adding support for `python3` and `cloud-init`

---

qemu

![buildroot booting](assets/buildroot-booting.png)

pstree

```text
init-+-acpid
     |-buildkitd---5*[{buildkitd}]
     |-containerd---7*[{containerd}]
     |-klogd
     |-sshd---sshd---sshd-+-bash---pstree
     |                    `-2*[sshfs---3*[{sshfs}]]
     |-supervise-daemo---lima-guestagent---3*[{lima-guestagent}]
     |-2*[supervise-daemo---getty]
     |-syslogd
     |-udevd
     `-udhcpc
```

```console
$ lima nproc
4
$ lima free -m
              total        used        free      shared  buff/cache   available
Mem:           3942          47        3761           3         133        3837
Swap:             0           0           0
$ lima df -h /
Filesystem                Size      Used Available Use% Mounted on
/dev/root                96.9G    269.4M     92.6G   0% /
$ lima sudo nerdctl version
Client:
 Version:	0.12.0
 Git commit:	<unknown>

Server:
 containerd:
  Version:	1.4.9
  GitCommit:
```

---

Building all the required software is done using [Buildroot](https://buildroot.org/).

The download is around 100 MB, with all batteries included.

```text
98M	output/disk.img.gz
```

See [https://github.com/afbjorklund/buildroot4containers](https://github.com/afbjorklund/buildroot4containers/tree/lima)

---

There is currently no public release of Bean OS available.

Written by Anders F Björklund <https://github.com/afbjorklund>

---

Green Beans image credit: Designed by macrovector / [Freepik](http://www.freepik.com)
