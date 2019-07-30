---
layout: post
title: Install Elixir and Erlang with asdf version manager
author: silva96
categories: [tools, elixir, erlang]
image: assets/images/install-elixir-and-erlang-with-asdf.jpg
---

If you come from _`Ruby on Rails`_ for example, it's very common to have _`rvm`_ to manage your _`Ruby`_ versions, but also you would probably need to install _`nvm`_ to manage versions for _`Node.js`_ too (since _`Rails`_ now has _`webpack`_ as the default assets pipeline).

This same story happens with _`Phoenix Framework`_, but even adding a third language to the stage: _`Erlang`_, _`Elixir`_ and _`Node.js`_

So how do we manage all involved languages versions with one single version manager?

> meet `asdf` version manager

You can install it following the [installation instructions](https://asdf-vm.com/#/core-manage-asdf-vm?id=install-asdf-vm)

After installing, simply install an Erlang version by running

```bash
# to add the erlang plugin
$ asdf plugin-add erlang https://github.com/asdf-vm/asdf-erlang.git
# to list all available erlang versions
$ asdf list-all erlang
```

A long list may appear, you usually would want to install the latest (at the bottom)

```bash
$ asdf install erlang 22.0.7
```

After installing Erlang, you can proceed to install Elixir with the same approach

```bash
$ asdf plugin-add elixir https://github.com/asdf-vm/asdf-elixir.git
$ asdf list-all elixir
$ asdf install elixir 1.4.9
```

Then, in your project folder, you can tell asdf to select specific versions of software by adding a `.tool-versions` file with some content like this:

```
elixir 1.9.1
erlang 22.0.7
nodejs 10.16.0
```

If you call `asdf current` to see whats the current versions of each configured plugin inside the current working directory

```
$ asdf current

  elixir         1.9.1    (set by /Users/benja/dev/newslettex/.tool-versions)
  erlang         22.0.7   (set by /Users/benja/dev/newslettex/.tool-versions)
  version 10.16.0 is not installed for nodejs
```

You notice that it says node 10.16.0 is not installed, because it reads the `.tool-versions` and finds out we don't have the requested nodejs version. To install it, simply:

```
$ asdf plugin-add nodejs https://github.com/asdf-vm/asdf-nodejs.git
$ brew install coreutils
$ brew install gpg
$ bash ~/.asdf/plugins/nodejs/bin/import-release-team-keyring
$ asdf install nodejs 10.16.0
```

Now you have all your versions managed under asdf.
