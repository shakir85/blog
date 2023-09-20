---
title: "Docker image from scratch"
description: "Understanding scratch base layer for building single-binary containers"
date: 2023-04-28T15:23:17-07:00
categories: ["docker"]
tags: ["how to"]
draft: false
---

Single binary containers provide a confined environment where nothing is in the container except the application and its runtime dependencies. Such approach reduces the attack surface and limits malicious application activity.

This is different from minimal containers. In minimal containers, you have a stripped-down version of a typical Linux distro containing only the minimum needed libraries, tools, and runtime to run a specific application or service.

A single-binary container is built using Docker's scratch image. The scratch base is a Docker's reserved *blank image* that acts like an empty layer to create parent images. A Docker scratch image has no filesystem, C libraries (not even `glibc`), or binaries.

Docker offers building from `scratch` images to give us complete control over what can be shipped inside the container. We can use them to build a complete distribution or just a single-binary application.

## Single binary container requirements

In a single binary container, we only need the application and its dependencies/libraries available in the container. In compiled languages, the application must be [statically compiled/built](https://en.wikipedia.org/wiki/Static_build).

Let's take a look at this simple hello-world C code

```c
#include <stdio.h>

int main(void) {
     printf("hello world");
     return 0;
}
``` 
  
Compile it using `--static` flag to include the required libraries with the compiled artifact

```sh
gcc -o hello --static hello.c
```

Create the `Dockerfile`

```dockerfile
FROM scratch
COPY hello /
CMD [ "/hello" ]
```

Build the image and run the container

```sh
docker build --no-cache -t my-scratch:latest .
docker run --rm my-scratch
```

If we try to send an `echo` command to the container, it will fail because there is no such a binary or application in the scratch container

```sh
docker run --rm my-scratch echo hi

docker: Error response from daemon: failed to create shim task: OCI runtime create failed: 
runc create failed: unable to start container process: exec: 
"echo": executable file not found in $PATH: unknown.
```

## More Complexity

Say, for example, we want to add the `echo` command to the scratch container. Since `echo` is a compiled binary, we may think we can copy it from another parent image into the scratch image using `COPY --from=ubuntu:latest /usr/bin/echo /` in the Dockerfile.

The catch is that since `echo` is a dynamically linked binary, it will need some dependencies in order to run. We can use the `ldd` command [^1] to view what library `echo` depends on.

Let's jump into an Ubuntu container and examine that

```sh
docker run -it --rm ubuntu:latest bash
root@cd3dd0afeb53:/# which echo
/usr/bin/echo

root@cd3dd0afeb53:/# ldd /usr/bin/echo
    linux-vdso.so.1 (0x00007ffe99d81000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fecf34c3000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fecf36f9000)
```

The output shows the `echo` command's dependencies. So basically, the `libc` library and a special object `vdso`, provided by the kernel, must be in the container; and without them, the `echo` command will not work.

## Conclusion

Single binary Docker containers are containers built using the `scratch` base image and stripped down to contain only the application and its dependencies, resulting in a smaller, more secure, and more lightweight container image. In some cases, this provides efficient deployment. However, it depends on the application's needs because handling the low-level runtime requirements can get complicated.

[^1]: This [Reddit post](https://www.reddit.com/r/linux/comments/ylg6rd/linux_instrumentation_part_4_ldd/?utm_source=share&utm_medium=web2x&context=3) shows some interesting facts about `ldd`.
