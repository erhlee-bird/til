---
date: '2024-08-09T18:15:23-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["elixir", "github-actions", "nix"]
title: 'Build Mix Releases With Nix Flakes And Private Packages'
---

We have a license to the fantastic Elixir [Oban Pro](https://getoban.pro/)
library. To retrieve the Oban Pro and Oban Web dependencies, we must add their
third-party Hex repository and authenticate with our auth key like so:

```shell
mix hex.repo add oban https://getoban.pro/repo \
  --fetch-public-key SHA256:4/OSKi0NRF91QVVXlGAhb/BIMLnK8NHcx/EWs+aIWPc \
  --auth-key ${OBAN_AUTH_KEY}
```

However, if we were to build our Elixir application and generate a `mix release`
using `nix build`, we would have to introduce build-time secrets. Some of the
approaches can be perused [here in the NixOS wiki](https://nixos.wiki/wiki/Comparison_of_secret_managing_schemes).

In this scenario, we instead opt to retrieve dependencies as a GitHub Actions
step outside of Nix's responsibility. We populate the `deps/` folder by invoking
`mix deps.get --only $MIX_ENV` and use the path to allow `nix build` to consider
files that aren't part of the git repository.

```yaml
jobs:
  nix-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: cachix/install-nix-action@v27
        with:
          extra_nix_config: |
            access-tokens = github.com=${{ secrets.GITHUB_TOKEN }}
            sandbox = false
          nix_path: "nixpkgs=channel:nixos-unstable"
      - run: |
          nix-shell -p cacert elixir --run '
            # ----- #
            set -eo pipefail

            mix local.hex --force
            mix local.rebar --force

            mix hex.repo add oban https://getoban.pro/repo \
              --fetch-public-key SHA256:4/OSKi0NRF91QVVXlGAhb/BIMLnK8NHcx/EWs+aIWPc \
              --auth-key "${{ secrets.OBAN_AUTH_KEY }}"

            mix deps.get --only $MIX_ENV
            # ----- #
          '
        env:
          ELIXIR_ERL_OPTIONS: "+fnu"
          MIX_ENV: "prod"
        shell: bash
      - run: nix build path://$(pwd)
```

Additionally, for local testing of the GitHub Actions workflow, we can use [act](https://github.com/nektos/act).

```shell
sudo $(which act) -W .github/workflows/nix-ci.yml -s GITHUB_TOKEN=$(gh auth token) -s OBAN_AUTH_KEY=$OBAN_AUTH_KEY
```
