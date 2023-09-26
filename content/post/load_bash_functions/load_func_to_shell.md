---
title: "Importing functions in shell scripting"
description: "How to load a function from a local or remote source to your current shell"
date: 2023-06-22T08:34:22-07:00
categories: ["linux", "shell"]
tags: ["how to", "good to know"]
draft: false
---

In this post, I will show you how to *source* (load) a Bash function from a local or remote source (e.g. a file in Github) into the current shell.

In shell scripting, using the `source` command (also known as the dot "`.`" command) allows to read and execute commands from a script file, and load its content into your current shell. This makes all variables, functions, and aliases defined in that script file become available in the current shell session.

## Load from local file

Similar to importing libraries in programming languages, you can organize your freqnetly used code in different files in your project directory and then load them as you need. See this example:

```text

# Project directory tree

root
├── lib/
│   └── common.sh
├── var/
│   └── stuff.sh
│
└── main_script.sh
```

If you want to import a function from `root/lib/common.sh` to `main_script.sh`, you only need to source that file:

```sh
#!/bin/bash
# *** main_script.sh ***
source lib/common.sh
# .
# code
# .
```

## Load from remote file

To load script content into your current shell without downloading the remote file, you can `curl` the content of the script and redirect it to a `source` command as the following (don't forget the `-s` flag to silence curl's download info):

```sh
# You can use a dot `.` instead of `source` as well
$ source <(curl -s https://raw.githubusercontent.com/shakir85/utils/main/print_hello)
```

Remote file content:

```sh
#!/bin/bash
print_hello() {
  echo "This is the boring hello world message"
}

```

After that, you can invoke the `print_hello` function from your current shell:

```sh
$ print_hello
This is the boring hello world message
```

This technique allows us to set environment variables, import functions, or modify the current shell's behavior using the contents of a remote script. This is a cool trick when you do not want to persist the data of the remote script on the host or want to load functions or variables to the current working shell on the fly. The downside, though, is that if the remote content vanishes, your script could bust!
