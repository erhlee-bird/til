---
date: '{{ .Date }}'
draft: false
params:
  author: "Eric Lee"
tags: []
title: '{{ replace .File.ContentBaseName `-` ` ` | title }}'
---

