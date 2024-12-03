---
date: '2024-12-03T16:38:16-05:00'
draft: false
params:
  author: "Eric Lee"
tags: []
title: 'Elixir Mix Reenable'
---

TIL, mix doesn't let you re-run the same task multiple times. If I want an alias,
for example, that creates and migrates multiple repos, I need to run
`Mix.Task.reenable` on the tasks that I am reusing.

```elixir
# mix.exs

...

  defp aliases do
    ...

    "ecto.setup": [
      "ecto.create -r MyApp.RepoA",
      "ecto.migrate -r MyApp.RepoA",
      ~s|run -e 'Mix.Task.reenable("ecto.create")'|,
      ~s|run -e 'Mix.Task.reenable("ecto.migrate")'|,
      "ecto.create -r MyApp.RepoB",
      "ecto.migrate -r MyApp.RepoB"
    ]
  end
...

```
