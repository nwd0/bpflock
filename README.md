# bpflock - Lock Linux machines

`bpflock` - eBPF driven security for locking and auditing Linux machines.

#### This is a Work In Progress:

* `bpflock` is currently in experimental stage and some BPF programs are being updated.

* Programs will be updated soon to use [Cilium ebpf library](https://github.com/cilium/ebpf/) and turned into a small daemon.

## Sections

* [1. Introduction](https://github.com/linux-lock/bpflock#1-introduction)
  - [1.1 Security features](https://github.com/linux-lock/bpflock#11-security-features)
  - [1.2 Semantics](https://github.com/linux-lock/bpflock#12-semantics)
* [2. Deployment]
* [3. Build](https://github.com/linux-lock/bpflock#3-build)


## 1. Introduction

`bpflock` combines multiple bpf independent programs to restrict access to a wide range of Linux features. Only programs like systemd, container managers or other containers that run in the host [pid namespace](https://man7.org/linux/man-pages/man7/namespaces.7.html) will be able to access all Linux kernel features, other tasks and containers will be restricted or completely blocked.

bpflock protects Linux machines using a system wide approach taking advantage of [LSM BPF](https://www.kernel.org/doc/html/latest/bpf/bpf_lsm.html).

Note: bpflock is able to restrict root access to some features, however it does not protect against evil root users.


## 1.1 Security features

`bpflock` bpf programs offer multiple security protections to restrict access to the following features:

* [Hardware additions](https://github.com/linux-lock/bpflock/tree/main/doc/hardware-additions.md)
  - [USB additions protection](https://github.com/linux-lock/bpflock/tree/main/doc/hardware-additions.md#1-usb-additions-protection)

* [Memory protections](https://github.com/linux-lock/bpflock/tree/main/doc/memory-protections.md)
  - [Kernel image lock down](https://github.com/linux-lock/bpflock/tree/main/doc/memory-protections.md#1-kernel-image-lock-down)
  - [Kernel modules protection](https://github.com/linux-lock/bpflock/tree/main/doc/memory-protections.md#2-kernel-modules-protections)
  - [BPF protection](https://github.com/linux-lock/bpflock/tree/main/doc/memory-protections.md#3-bpf-protection)
  - [Execution of Memory ELF binaries](https://github.com/linux-lock/bpflock/tree/main/doc/memory-protections.md#4-execution-of-memory-elf-binaries)

* [Filesystem protections](https://github.com/linux-lock/bpflock/tree/main/doc/filesystem-protections.md)

  - Read-only root filesystem protection
  - sysfs protection

* [Namespaces protections](https://github.com/linux-lock/bpflock#34-namespaces-protections)

### 1.2 Semantics

The semantic of all features is:

* Permission: each program supports three different permission models.
  - `allow|none` : access is allowed.
  - `restrict` : access is allowed only from processes that are in the initial pid namespace.
  - `deny` : access is denied for all processes.

* Allowed or blocked operations/commands:
  when a program runs under the `allow` or `restrict` permission model, a list of allowed or blocked commands can be specified with:
  - `allow` : comma-separated list of allowed operations.
  - `block` : comma-separated list of blocked operations.


## 2. Deployment

bpflock needs a `5.15` with the following configuration:

```code
CONFIG_DEBUG_INFO=y
CONFIG_DEBUG_INFO_BTF=y
CONFIG_KPROBES=y
CONFIG_LSM="...,bpf"
CONFIG_BPF_LSM=y
```


* [Docker deployment](https://github.com/linux-lock/bpflock/blob/master/doc/deploy-docker.md)


## 3. Build

bpflock uses [docker BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/) to build and [Golang](https://go.dev/doc/install) for running tests.



### 3.1 libbpf

This repository uses libbpf as a git-submodule. After cloning this repository you need to run the command:

```bash
git submodule update --init
```

If you want submodules to be part of the clone, you can use this command:

```bash
git clone --recurse-submodules https://github.com/linux-lock/bpflock
```

### 3.2 Libraries and compilers

#### Ubuntu

To build install the following packages:
  ```bash
  sudo apt install -y bison build-essential flex \
        git libllvm10 llvm-10-dev libclang-10-dev \
        zlib1g-dev libelf-dev libfl-dev
  ```

### 3.3 Build binaries

Get libbpf if not:
```
git submodule update --init
```

To build just run:
```bash
make
```

All build binaries and libraries will be produced in `build/dist/` directory.

Current build process was inspired from: https://github.com/iovisor/bcc/tree/master/libbpf-tools
