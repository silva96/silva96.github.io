---
layout: post
title: Continuous Integration in Elixir Phoenix with Travis-CI and Codecov
author: silva96
categories: [elixir, phoenix, testing, continuous-integration, travis, coverage, codecov]
image: assets/images/continuous-integration-phoenix-travis.jpg
---
I'll show how to get travis-ci run your test suit and then upload the coverage report to codecov. All this, being shown on github pull request checks for reference.

Something I've been doing more extensively lately is Unit Testing, not necessarily doing TDD but Unit testing per se. When you start enjoying it it becomes like a challenge and also an addiction, getting you coverage reporting tool show a 100% coverage badge in your project is a big joy. Unit Testing also makes you produce better code and you feel safer, so it's something positive to add to your development process.

### Hands on

First, you will have to register at [travis-ci.org](https://travis-ci.org) and [codecov.io](https://codecov.io) both have free plans for public repositories. (sign up using github)

Select the repositories you want to integrate in both services like shown above:


![Travis select repositories](/assets/images/posts/select-repos.png)

For the reporting tool we will be using the package `ExCoveralls`, add the following to your `mix.exs` file in your Phoenix project

```elixir
defmodule Yourapp.MixProject do
  use Mix.Project

  def project do
    [
      app: :yourapp,
      version: "0.1.0",
      elixir: "~> 1.5",
      elixirc_paths: elixirc_paths(Mix.env()),
      compilers: [:phoenix, :gettext] ++ Mix.compilers(),
      start_permanent: Mix.env() == :prod,
      aliases: aliases(),
      deps: deps(),
      test_coverage: [tool: ExCoveralls] # ADD THIS LINE
    ]
  end

  ...

  defp deps do
    [
      {:phoenix, "~> 1.4.9"},
      {:phoenix_pubsub, "~> 1.1"},
      {:phoenix_ecto, "~> 4.0"},
      {:ecto_sql, "~> 3.1"},
      {:postgrex, ">= 0.0.0"},
      {:phoenix_html, "~> 2.11"},
      {:excoveralls, "~> 0.11.1"}, # ADD THIS LINE
      {:phoenix_live_reload, "~> 1.2", only: :dev},
      {:gettext, "~> 0.11"},
      {:jason, "~> 1.0"},
      {:plug_cowboy, "~> 2.0"}
    ]
  end

  ...

end
```

Also create a `.travis.yml` file in the root folder of your project, as follows:

```yaml
language: elixir

elixir:
  - '1.9.1'
otp_release:
  - '22.0.7'

env:
  - MIX_ENV=test

addons:
  postgresql: '9.6'

services:
  - postgresql

before_script:
  - cp config/travis.exs config/test.exs
  - mix do ecto.create, ecto.migrate

script:
  - mix do compile --warnings-as-errors, coveralls.json

after_success:
  - bash <(curl -s https://codecov.io/bash)

cache:
  directories:
    - _build
    - deps
```

The before_script will replace a `test.exs` file for the `travis.exs` so it will run an environment with specific things for travis to work properly.

So we need to create this `travis.exs` under `config` folder

```elixir
use Mix.Config

# Configure your database
config :yourapp, yourapp.Repo,
  username: "postgres",
  password: "",
  database: "yourapp_test",
  hostname: "localhost",
  pool: Ecto.Adapters.SQL.Sandbox

# We don't run a server during test. If one is required,
# you can enable the server option below.
config :yourapp, yourappWeb.Endpoint,
  http: [port: 4002],
  server: false

# Print only warnings and errors during test
config :logger, level: :warn
```

You will also want to configure the coverage reporting tool to exclude some files we are not going to test nor take in account for our coverage metrics

create a `coveralls.json` file in the root folder of the project

```json
{
  "coverage_options": {
    "minimum_coverage": 100
  },
  "skip_files": [
    "test/",
    "lib/yourapp_web.ex",
    "lib/yourapp.ex",
    "lib/yourapp/application.ex",
    "lib/yourapp/repo.ex",
    "lib/yourapp_web/endpoint.ex",
    "lib/yourapp_web/gettext.ex",
    "lib/yourapp_web/views/error_helpers.ex"
  ]
```

This minimum_coverage config will mark as fail a build that doesn't have 100% coverage, which is hard to achieve in a real life project (but not impossible!).

to show badges of the build passing and coverage percentage like this:

![Github Badges](/assets/images/posts/badges.png)

add this, replacing yourgithub with your github username and yourrepo with the repository name

```markdown
[![codecov](https://codecov.io/gh/yourgithub/yourrepo/branch/master/graph/badge.svg)](https://codecov.io/gh/yourgithub/yourrepo)
[![Build Status](https://travis-ci.org/yourgithub/yourrepo.svg?branch=master)](https://travis-ci.org/yourgithub/yourrepo)
```

With all this configuration, every time you create a pull request you will see something like this:
![Github PR coverage comment1](/assets/images/posts/1-pr-ci.png){: .center}
![Github PR coverage comment2](/assets/images/posts/2-pr-ci.png){: .center}

and the PR list will show checks or crosses depending on the check status

![Github PR checks with travis-ci](/assets/images/posts/github-pr-checks.png)
