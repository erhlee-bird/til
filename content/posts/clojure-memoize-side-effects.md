---
date: '2024-07-25T02:05:39-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["clojure"]
title: 'Clojure Memoize Side Effects'
---

The following is a trick to make a function only exhibit its side effects once.

```clojure
(def warn-once
  (memoize
    (fn []
      (prn "Hello Warning"))))

(warn-once)
;; "Hello Warning"
(warn-once)
```
