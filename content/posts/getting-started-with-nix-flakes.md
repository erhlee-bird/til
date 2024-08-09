---
date: '2024-08-09T16:39:24-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["nix"]
title: 'Getting Started With Nix Flakes'
---

> Nix flakes provide a standard way to write Nix expressions (and therefore packages) whose dependencies are version-pinned in a lock file, improving reproducibility of Nix installations.
>
> - A flake refers to a file-system tree whose root directory contains the Nix file specification called flake.nix.

https://nixos.wiki/wiki/Flakes

## Generate a flake.nix file

To begin with, you need to generate a `flake.nix` file.

```shell
# Generate a very basic flake.nix.
nix flake init

# Generate a flake.nix from a template.
nix flake init --template "https://flakehub.com/f/the-nix-way/dev-templates/*#shell"
```

## Allow Unfree

In order to allow the inclusion of unfree packages, you can set a config
setting `config.allowUnfree = true;`.

To set it simply for a flake, you can add it to a `nixpkgs` input.

```nix
{
  ...

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; config.allowUnfree = true; };
      in
        ...
}
```

## Git-related gotchas

> For flakes in git repos, only files in the working tree will be copied to the store.

If you have a repo with dependencies for example that might have been added to
a `.gitignore` file, these files will not be added to the store and will not be
available during any `nix build` runs.

**If** you need to circumvent this limitation and reproducibility is not your
primary goal, you can invoke `nix build` with a path to include everything, even
files not tracked by git.

```shell
nix build path://$(pwd)
```
