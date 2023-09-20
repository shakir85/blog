---
title: "Importing functions in shell scripting"
description: "How to load a function from a local or remote source to your current shell"
date: 2023-06-22T08:34:22-07:00
categories: ["linux", "shell"]
tags: ["how to", "good to know"]
draft: false
---

In this post, I will show you how to load a Bash function from a local file and a remote source (e.g. a file in Github) into the current shell.

In shell scripting, using the `source` command (also known as the dot "`.`" command) you can  read and execute commands from a script file, and load its content into your current shell environment without spawning a new subshell.

When the `source` command is used, the script file is read and executed line by line, and any variables, functions, or aliases defined in the script become available in the current shell session.

## Load local functions to current shell

This operates similarly to the way imports work in typical programming languages. To organize your script, you'd typically store frequently used functions in a folder, for example, named lib. Subsequently, you can import these functions as shown below:

Project directory tree

```text
root
├── lib/
│   └── common.sh
├── var/
│   └── stuff.sh
│
└── mainscript.sh
```

If you want to import a function from `root/lib/common.sh` to `mainscript.sh`, you only need to source that file:

```sh
#!/bin/bash
# *** mainscript.sh ***
source lib/common.sh
# .
# code
# .
```

## Load remote functions to current shell

You can load a script's content into your shell without downloading the remote file by curling the content of the script and redirecting it to a `source` command as the following:

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

## Why would someone import a remote script?

This technique allows us to set environment variables, import functions, or modify the current shell's behavior using the contents of a remote script. This is a cool trick when you do not want to persist the data of the remote script on the host or want to load functions or variables to the current working shell on the fly. The downside, though, is that if the remote content vanishes, your script could bust!
