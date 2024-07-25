---
date: '2024-07-25T02:11:02-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["clojure"]
title: 'Clojure Namespace Shenanigans'
---

# Juxtaposing Functions in a Namespace

If you had a Clojure namespace with a bunch of functions that followed a
particular naming convention like `plugin-foo`, `plugin-bar`, etc. and wanted
to apply them all to an input, you could do the following.

```
(defn plugin-foo [arg] ...)
(defn plugin-bar [arg] ...)
(defn plugin-baz [arg] ...)

(defmacro juxt-all
  "Create a juxt function using all functions found by the provided prefix in
   the specified namespace."
  [ns prefix]
  `(fn [& args#]
     (let [fs# (keep (fn [[k# v#]]
                       (when (clojure.string/starts-with k# ~prefix)
                         v#))
                     (ns-publics ~ns))]
       (reduce (fn [acc# f#]
                 (into acc# (apply f# args#)))
               []
               fs#))))

(def ^:const juxt-plugins
  (juxt-all (symbol (namespace ::this))
            "plugin-"))

(juxt-plugins {:key "value"})
```

> **_NOTE:_** `ns-publics` doesn't guarantee any particular order.

```
(def foo "hi")
#'user/foo
(ns-publics (symbol (namespace ::this)))
{foo #'user/foo}
(def bar "world")
#'user/bar
(ns-publics (symbol (namespace ::this)))
{bar #'user/bar, foo #'user/foo}
```

# Sequentially running a series of functions in the current namespace

We run a custom function that parses the source file and extracts the list of
functions sought in order as again `ns-publics` does not preserve order.

```
(defn ns-publics-in-order
  "This is a hack. `ns-publics` does not return functions in source-order.
   By loading the source text and filtering for function definitions ourselves,
   we can provide that alternative."
  [ns]
  (some->> ns
           ns-publics
           vals
           first
           meta
           :file
           (.getResourceAsStream (RT/baseLoader))
           clojure.java.io/reader
           line-seq
           (filter #(and (clojure.string/starts-with? % "(defn")
                         (clojure.string/includes? % "-hook")))
           ;; Assumption is that all function definitions have a leading `defn-`
           ;; followed by the name and possible a spec.
           ;;
           ;; (defn `name`
           ;;   ...)
           ;;
           ;; (defn-spec `name` `spec`
           ;;   ...)
           (map #(clojure.string/split % #" "))
           (map second)
           (map symbol)
           (map (fn [x] [x (ns-resolve ns x)]))
           (into (flatland.ordered.map/ordered-map))))

(defmacro reduce-all
  "Sequentially run a series of functions found by the provided suffix in the
   current namespace."
  [ns suffix]
  (let [publics (ns-publics-in-order (eval ns))]
    `(fn [state# short-circuit-fn#]
       (let [fs# (keep (fn [[k# v#]]
                         (when (clojure.string/ends-with? k# ~suffix)
                           v#))
                       ~publics)]
         (reduce (fn [acc# f#]
                   (if (short-circuit-fn# acc#)
                     acc#
                     (f# acc#)))
                 state#
                 fs#)))))

(defn foo-hook [state] ...)
(defn bar-hook [state] ...)
(defn baz-hook [state] ...)

;; NB: It is really important that this is a macro with an eval within.
;;     It captures and analyzes the source at compile time since the source
;;     might not be available from within an Uberjar context.
(defmacro reduce-hooks
  []
  (eval '(til.core/reduce-all (symbol (namespace ::this)) "-hook")))
```
