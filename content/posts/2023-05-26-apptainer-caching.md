+++
title = "Apptainer cache validity"
date = 2023-05-26T16:50:00Z
tags  = ["apptainer", "docker", "podman"]
featured_image = ""
description = ""
aliases = [
    "/posts-output/2023-05-26-apptainer-caching/",
]
+++

I was recently asked whether [Apptainer][apptainer] (forked from the Singularity project) will
definitely update its cache of Apptainer images created from OCI/Docker images if a tagged image changes on a remote repo.
Let's find out:

Let's start by creating a v. simple image that just sends a message to stdout when executed.
After creating the image we tag it so we have an identifier to push to a remote image registry.

```
$ mkdir dockertest
$ cd dockertest

$ cat > Dockerfile <<EOF
FROM alpine:latest
CMD ["echo", "original"]
EOF

$ podman build -t willfurnasstuos/dockertest:latest .
...
```

Here I used [Podman][podman] to build the image but could equally have used Docker 
(I just prefer working with Podman, the reasons for which are best saved for another post).

Let's push that tagged image to `hub.docker.com`:

```
$ podman login
...
$ podman push willfurnasstuos/dockertest:latest
```

Now, if we run it we unsurprisingly get:

```
$ apptainer run docker://willfurnasstuos/dockertest:latest
...
original
```

Behind the scenes Apptainer has pulled that image from `hub.docker.com` and
converted it from OCI/Docker format to Apptainer's own format before executing it.
Cached OCI/Docker image layers and cached Apptainer images created by the on-the-fly conversion from OCI/Docker images are
cached under `~/.apptainer/cache`.

Now, let's see what Apptainer does if the tagged image on `hub.docker.com` is updated:

```
$ cat > Dockerfile <<EOF
FROM alpine:latest
CMD ["echo", "updated"]
EOF

$ podman build -t willfurnasstuos/dockertest:latest .
...
$ podman push willfurnasstuos/dockertest:latest
...

$ apptainer run docker://willfurnasstuos/dockertest:latest
...
updated

podman logout
```

It seems Apptainer automatically recognised that the remote image had been updated so updated its cache.  Woo!

[apptainer]: https://apptainer.org/
[podman]: https://podman.io/
