---
date: '2024-07-25T01:51:23-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["git", "hugo"]
title: 'Git Patching Hugo Themes'
---

Let's say you have a Hugo theme that you are using that has open PRs to fix
little issues like the theme not rendering when hosted from a subpath.

You could manage a fork of the repo or you could save off the PR as a git
patch to apply during CI/CD.

```shell
cd themes/mini
# make your changes to the submodule
git diff > ../mini-pr-132.patch
```

Once you've committed the patch to your repo, update your CI/CD to apply it
after checking the repo out.

```yaml
- name: Checkout
  uses: actions/checkout@v4
  with:
    submodules: recursive
    fetch-depth: 0
- name: Apply Patches
  run: |
    cd themes/mini && git apply ../mini-pr-132.patch
```
