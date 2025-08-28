---
date: '2025-08-28T12:22:11-04:00'
draft: false
params:
  author: "Eric Lee"
tags: ["bash"]
title: 'Bash Concurrency Job Semaphore'
---

The following two scripts should roughly express the same intent.
Run up to 8 jobs in parallel that print a number from a sequence.

```shell
$ seq 1000 | START="$(date +%s%3N)" parallel -j 8 'export NOW=$(date +%s%3N); echo "[$((NOW-START))ms] {}"; sleep 1'
[96ms] 1
[98ms] 2
[100ms] 3
[102ms] 4
[104ms] 5
[106ms] 6
[108ms] 7
[111ms] 8
[1106ms] 9
[1108ms] 10
[1109ms] 11
[1111ms] 12
[1115ms] 13
[1117ms] 14
[1118ms] 15
[1119ms] 16
[2119ms] 17
[2121ms] 18
[2123ms] 19
[2125ms] 20
[2128ms] 21
[2131ms] 22
[2134ms] 23
[2137ms] 24
[3132ms] 25
[3134ms] 26
[3136ms] 27
[3139ms] 28
[3141ms] 29
[3143ms] 30
[3144ms] 31
[3147ms] 32
```

```shell
#!/usr/bin/env bash
# ./demo.sh

START="$(date +%s%3N)"

N="${N:-8}"
for i in {1..1000}; do
  pids=( $(jobs -pr) )
  [[ ${#pids[@]} -ge ${N} ]] && wait -n "${pids}"

  {
    NOW="$(date +%s%3N)"
    echo "[$((NOW-START))ms] $i"
    sleep 1
  } &
done
```

```shell
$ ./demo.sh
[5ms] 1
[6ms] 2
[7ms] 3
[8ms] 4
[9ms] 5
[9ms] 6
[10ms] 7
[11ms] 8
[1012ms] 9
[1013ms] 10
[1014ms] 11
[1015ms] 12
[1015ms] 13
[1016ms] 14
[1019ms] 15
[1020ms] 16
[2018ms] 17
[2019ms] 18
[2020ms] 19
[2020ms] 20
[2021ms] 21
[2022ms] 22
[2025ms] 23
[2026ms] 24
[3025ms] 25
[3025ms] 26
[3026ms] 27
[3027ms] 28
[3027ms] 29
[3029ms] 30
[3030ms] 31
[3031ms] 32
```
