[![Build Status](https://badge.buildkite.com/9e0e6c88972a3248a0908506d6946624da84e4e18c0870c4d0.svg)](https://buildkite.com/rust-vmm/kvm-ci)

# `kvm` - Rust Bindings and Wrappers Linux's Kernel Virtual Machine APIs

This repository contains the `kvm-bindings` and `kvm-ioctls` crates.

## kvm-ioctls

[![crates.io](https://img.shields.io/crates/v/kvm-ioctls.svg)](https://crates.io/crates/kvm-ioctls)

The kvm-ioctls crate provides safe wrappers over the
[KVM API](https://www.kernel.org/doc/Documentation/virtual/kvm/api.txt), a set
of ioctls used for creating and configuring Virtual Machines (VMs) on Linux.
The ioctls are accessible through four structures:
- `Kvm` - wrappers over system ioctls
- `VmFd` - wrappers over VM ioctls
- `VcpuFd` - wrappers over vCPU ioctls
- `DeviceFd` - wrappers over device ioctls

For further details check the
[KVM API](https://www.kernel.org/doc/Documentation/virtual/kvm/api.txt) as well
as the code documentation.

## kvm-bindings

[![crates.io](https://img.shields.io/crates/v/kvm-bindings.svg)](https://crates.io/crates/kvm-bindings)

Rust FFI bindings to KVM, generated using
[bindgen](https://crates.io/crates/bindgen). It currently has support for the
following target architectures:
- x86\_64
- arm64
- riscv64

The bindings exported by this crate are statically generated using header files
associated with a specific kernel version, and are not automatically synced with
the kernel version running on a particular host. The user must ensure that
specific structures, members, or constants are supported and valid for the
kernel version they are using. For example, the `immediate_exit` field from the
`kvm_run` structure is only meaningful if the `KVM_CAP_IMMEDIATE_EXIT`
capability is available. Using invalid fields or features may lead to undefined
behaviour.

This crate also offers safe wrappers over FAM structs - FFI structs that have
a Flexible Array Member in their definition.
These safe wrappers can be used if the `fam-wrappers` feature is enabled for
this crate. Note that enabling the `fam-wrappers` feature enables the `vmm-sys-util`
dependency.

### Serialization

The crate also has an optional dependency on [`serde`](serde.rs) when enabling the
`serde` feature, to allow serialization of bindings. Serialization of
bindings happens as opaque binary blobs via [`zerocopy`](https://google.github.io/comprehensive-rust/bare-metal/useful-crates/zerocopy.html).
Due to the kernel's ABI compatibility, this means that bindings serialized
in version `x` of `kvm-bindings` can be deserialized in version `y` of the
crate, even if the bindings have had been regenerated in the meantime.

### Regenerating Bindings

Please see [`CONTRIBUTING.md`](kvm-bindings/CONTRIBUTING.md) in the `kvm-bindings` 
crate for details on how to generate the bindings or add support for new architectures.

## Supported Platforms

The kvm-ioctls can be used on x86\_64, aarch64 and riscv64 (experimental).

## Running the tests

Our Continuous Integration (CI) pipeline is implemented on top of
[Buildkite](https://buildkite.com/).
For the complete list of tests, check our
[CI pipeline](https://buildkite.com/rust-vmm/kvm-ci).

Each individual test runs in a container. To reproduce a test locally, you can
use the dev-container on x86\_64, arm64 and riscv64.

```bash
# For running riscv64 tests, replace v47 with v47-riscv. This provides an
# emulated riscv64 environment on a x86_64 host.
docker run --device=/dev/kvm \
           -it \
           --security-opt seccomp=unconfined \
           --volume $(pwd)/kvm:/kvm \
           rustvmm/dev:v47
cd kvm
cargo test
```

For more details about the integration tests that are run for `kvm`,
check the [rust-vmm-ci](https://github.com/rust-vmm/rust-vmm-ci) readme.
