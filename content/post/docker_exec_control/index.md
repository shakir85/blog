---
title: "Understanding CMD and ENTRYPOINT differences" 
description: "and how to hand over execution-flow from `ENTRYPOINT` to `CMD` like a boss"
date: 2023-07-16T17:34:13-07:00
categories: ["linux", "docker"]
tags: ["grasping essentials"]
draft: false
# slug: hello-world
# image: cover.jpg
# weight: 1
---

In this post, we will demonstrate how `ENTRYPOINT` and `CMD` work together, their differences, and how to redirect the runtime execution flow from `ENTRYPOINT` to the `CMD` where the main application's command is executed.

## The way `ENTRYPOINT` and `CMD` work together

In most cases, `CMD` and `ENTRYPOINT` instructions can be used interchangeably. Also, you do not have to use both of them in every Dockerfile you develop. However, each instruction offers additional features that can help you control how you want to initialize your container and run your application.


- `ENTRYPOINT` is like the "main command" or the starting point for your container. It's like the default action the container takes when you run it. For example, you might use `ENTRYPOINT` to start a web server or run a specific application.

- `CMD` is like providing additional arguments or options to the command specified in `ENTRYPOINT`. It's like saying, "When you start the container using the `ENTRYPOINT` command, here are some extra things to do". CMD is often used to pass *default* arguments to `ENTRYPOINT`. Note that we mentioned 'default arguments,' which we'll explain in the following example.

So, `ENTRYPOINT` sets the main command of the container, and `CMD` provides default arguments to that command. Here is an example: 

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello world"]
```

In this Dockerfile, the content of the `CMD` instruction is passed to the `ENTRYPOINT` as the default argument, so when we build and run the container without arguments it will print "Hello world":

```sh
docker build -t test .
docker run test
```

Output:
```test
Hello world
```
### Overriding `CMD`

Whether you are using `CMD` in combination with `ENTRYPOINT` or using it alone, you can override it as follows:

```sh
docker run test 'another hello world'
```

Output:
```text
another hello world
```

### Overriding `ENTRYPOINT`

Given this Dockerfile:

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello world"]
```
When you have a Dockerfile with only an `ENTRYPOINT` instruction, you need to use the  `--entrypoint` flag to override the entry-point command as the following:

```sh
docker docker run --entrypoint 'printenv' test
```

Output:

```text
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=7ffd59696373
HOME=/root
```

If you run the container while supplying a command without specifying the `--entrypoint` flag, Docker will treat the supplied command as additional arguments to the command specified in the `ENTRYPOINT`:

```sh
docker docker run test 'printenv'
```

Output:

```text
Hello world printenv
```

This is similar to having this entry-point in your Dockerfile: `ENTRYPOINT ["echo", "Hello world", "printenv"]`


## Handing over execution flow from `ENTRYPOINT` to `CMD`

Consider the following Python app:

```dockerfile
FROM python:latest
# ...
# RUN >>> install Python packages & configs ...
# COPY >>> add files and executables
# ...
ENTRYPOINT ["uvicorn"]
CMD ["main:app", "--host", "0.0.0.0"]
```

The image built from this Dockerfile is perfectly fine. The uvicorn command will be executed when the container runs, and the `CMD` instruction will provide the necessary arguments for the uvicorn server. 

However, in most cases, we require a method to incorporate runtime configurations that our Flask app needs during execution.

### Enter "docker-enterypoint.sh"

When developing a Dockerfile, it is a common pattern to wrap various initialization commands within a shell script, typically named "docker-entrypoint.sh", and execute it using an `ENTRYPOINT` instruction. The purpose of this technique is to provide a flexible way of configuring the Docker container *at runtime*. Typical runtime configuration tasks include: starting a service, exporting environment variables, or simply editing certain configuration files.

Since `ENTRYPOINT` instruction provides run-time execution, we need to find a way to return the execution flow back the Dockerfile's `CMD` instruction to run the main application command. 

To do so, simply add an `exec "@$"` statement at the very end of the shell script that is being executed by the `ENTRYPOINT` (`docker-entrypoint.sh`) file.

After adding all configuration scripts to 'docker-entrypoint.sh,' we will modify the Dockerfile as follows:

```dockerfile
FROM python:latest
# ...
# RUN >>> install Python packages & configs ...
# COPY >> add our app
# ....

# Copy the init script file to a directory in the PATH
# You might need to `chmod +x` it too
COPY docker-entrypoint.sh /usr/local/bin 

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

To visualize the process:

!["Dockerfile example"](visual1.png)

When we run the container, Docker will execute the `ENTRYPOINT`, which contains the "docker-entrypoint.sh" script. Then, the `exec "$@"` command in the "docker-entrypoint.sh" script will, in a sense, return control to the `CMD`. To clarify, the exec part won't transfer execution flow; it just expands the arguments specified in the `CMD` instruction (which can be overridden when using `docker run`).

Let's break down what `exec "@$"` does:

- `exec` is a Linux command used to replace the current process with a new process. In this case, it ensures that `"$@"` becomes the main process running in the container.
- `"$@"` expands to all the command-line arguments passed to the container when it starts (e.g. expanding the content of the `CMD` instruction). It preserves the exact arguments that were passed during container runtime. Also, you still can override the `CMD` by specifying args on the `docker run` command. Finally, note that you cannot place any commands in the 'docker-entrypoint.sh' file after the exec "$@" line.
