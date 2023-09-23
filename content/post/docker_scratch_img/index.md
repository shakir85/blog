---
title: "A practical approach for using Docker scratch base layer"
description: "When minimalism meets practicality"
date: 2023-04-28T15:23:17-07:00
categories: ["docker"]
tags: ["how to"]
draft: false
---

The scratch base is a Docker's reserved *blank image*, or an empty filesystem, that acts like an empty layer to create parent images. It is like an empty canvas. It's where you start building containers from scratch (no pun intended!), adding only what your application needs, making it super minimal. This gives us a complete control over what can be shipped inside the container.

In this post, I will show you two distinctive ways to utilize the 'scratch' base. The first part will explore how to create minimal Docker images primarily for sharing files with other images and use container hubs, like ECR and Docker Hub, as file storage. In the second part, I will discuss the advantages of using the scratch base layer for deploying single-binary applications.

## Sharing files between images

When building images, Docker gives us the ability to pull files from other images (remote or local) using the `--from=` option with the `COPY` instruction in Dockerfile. What's neat about this is that it enables us to cherry-pick specific files from another image and toss them into our new image while it's building. And the cherry on top? You can even pick files from a specific image by specifying in its tag. So if you have two tags for the image foo: `foo:latest` and `foo:1.2``, you can pull files from the version 1.2 on the fly. See this Dockerfile:

```dockerfile
FROM ubuntu:latest

COPY --from=foo:1.2 /content /content

# Other build commands ...
```

### Treat your container hub as a remote storage

Since we can copy files from remote images into a Dockerfile to include them in new image builds, we can actually store project files in the container registry as container images. You might wonder, why would you do that? Why not just use object storage like AWS S3 or even a Git repo to store and fetch files dynamically?

Well, it's just an additional option that comes with its own set of benefits:

1. You don't need to fuss with remote storage authentication (e.g., S3, Git, Artifactory, etc.), especially if you're already logged in to your container registry. You're already authenticated, which is super handy in CI/CD pipelines.

2. It brings reproducibility to the table. Every image in your pipeline can fetch files from a single source (image) that the pipeline is already has access to. This consistency makes it a breeze to replicate builds.

But, be aware that not planning how you use this approach can turn it into a dependency hill, and you might end up shooting yourself in the foot. So, use it wisely and be sure to document your approach!

So, if your intention is to use Docker images solely for storing files, then here's the approach you should take:

Dockerfile content:
```dockerfile
FROM scratch
# Copy all files you want to share from other images
COPY somescript.sh /content
COPY somearchive.tar.gz /content
# ...
```

Build the image:

```sh
docker build -t foo:1.2 .
```

Push to remote (skip if you want the image to be local)

```sh
docker push shakir85/foo:1.2
```

Then, copy the files from the remote container registry into your Docekrfile:

```dockerfile
FROM ubuntu:latest

# Syntax: COPY --from=<remote_repo>/<image_name>:<tag>
COPY --from=shakir85/project_files:latest /content /content
# ...
```

Although you can achieve the same result with minimal images like Alpine or Busybox, using a distro-based image solely for file storage and sharing in Docker is not as efficient as using scratch base image.

## Use sctach base for single binary containers

The scratch base layer can be an excellent choice for creating single-binary containers when your application and its dependencies are entirely self-contained within a single executable file. 

When you use `FROM scratch`, you start with an empty filesystem, and you can add only what is absolutely necessary for your application to run. This approach can help produce minimal container with a very small footprint because it contains only your application binary and nothing else. 

The catch is that, since the scratch layer is essentially an empty filesystem, your application must be [statically compiled](https://en.wikipedia.org/wiki/Static_build). Also, keep in mind that because your application is going to be statically compiled, a small-sized container is not guaranteed. The container's size really depends on the type and requirements of the application and the number of libraries or dependencies that need to be included (compiled) along with the application.

That being said, let's take a look at this simple hello-world C code:

```c
#include <stdio.h>

int main(void) {
     printf("hello world");
     return 0;
}
``` 
  
Compile it using `--static` flag to include the required libraries:

```sh
gcc -o hello --static hello.c
```

Create the `Dockerfile`:

```dockerfile
FROM scratch
COPY hello /
CMD [ "/hello" ]
```

Build the image and run the container:

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

### Bonus

Say, for example, we want to add the `echo` command to the scratch container. Since `echo` is a compiled binary, we may think we can copy it from another parent image into the scratch image using `COPY --from=ubuntu:latest /usr/bin/echo /` in the Dockerfile.

However, since `echo` is a dynamically linked binary, the `echo` binary will need some dependencies in order to run. We can use the `ldd` command [^1] to view what libraries `echo` depends on. Let's jump into an Ubuntu container and examine that:

```sh
docker run -it --rm ubuntu:latest bash
root@cd3dd0afeb53:/# which echo
/usr/bin/echo

root@cd3dd0afeb53:/# ldd /usr/bin/echo
    linux-vdso.so.1 (0x00007ffe99d81000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fecf34c3000)
    /lib64/ld-linux-x86-64.so.2 (0x00007fecf36f9000)
```

The output shows the `echo` command's dependencies that must be in the container, which without them, the `echo` command will not work.


[^1]: This [Reddit post](https://www.reddit.com/r/linux/comments/ylg6rd/linux_instrumentation_part_4_ldd/?utm_source=share&utm_medium=web2x&context=3) shows some interesting facts about `ldd`.
