---
title: Nix-ld
description: A Solution for Issues with Pre-Compiled Executables on NixOS
draft: false
date: 2024-01-03
tags:
  - nixos
  - bugfix
  - nix-ld
---

_No such file or directory: How I stopped worrying and started loving binaries_
_on NixOS._

In this article, I will discuss the technical issue of running pre-compiled
executables on NixOS, and how we can improve the user experience by making these
binaries work seamlessly using [nix-ld](https://github.com/Mic92/nix-ld).

One of the key benefits of [NixOS](https://nixos.org/) is its focus on purity
and reproducibility. The operating system is designed to ensure that the system
configuration and installed software are always in a known and predictable
state. This is achieved through the use of the Nix package manager, which allows
users to declaratively specify their system configuration and software
dependencies.

However, this focus on purity can make it difficult for users to run
pre-compiled executables that were not specifically designed for NixOS. These
executables may have dependencies on libraries that are not available in the Nix
package manager, or may require patching or modification to work correctly on
the operating system.

```nix
{ config, pkgs, ... }: {
  # Enable nix ld
  programs.nix-ld.enable = true;

  # Sets up all the libraries to load
  programs.nix-ld.libraries = with pkgs; [
    stdenv.cc.cc
    zlib
    fuse3
    icu
    nss
    openssl
    curl
    expat
    # ...
  ];
}

```

## Conclusion

In conclusion, nix-ld is a useful tool for running pre-compiled executables on
NixOS without the need for patching or modification. It provides a layer
that allows users to specify the necessary libraries for each executable and
improves the user experience by allowing users to easily run binaries from
third-party sources and proprietary software. By including the most common
libraries in the NixOS configuration, nix-ld can provide an even more seamless
experience for running pre-compiled executables on NixOS.
