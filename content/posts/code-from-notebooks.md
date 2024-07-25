---
date: '2024-07-25T02:03:38-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["bash", "jupyter"]
title: 'Code From Notebooks'
---

```
cat notebook.ipynb | jq -r '.cells[] | select(.cell_type == "code") | .source[]' | paste - -
```
