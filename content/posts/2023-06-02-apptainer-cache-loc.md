+++
title = "Apptainer cache location"
date = 2023-06-02T12:50:00Z
tags  = ["apptainer"]
featured_image = ""
description = ""
aliases = [
    "/posts-output/2023-06-02-apptainer-cache-loc/",
]
+++

I was trying to figure out where [Apptainer][apptainer] caches images in its SIF format created on the fly from OCI images pulled from a remote registry.

The location appears to be under `~/.apptainer/cache/oci-tmp/`, as can be seen if you explicitly convert a Docker image to a SIF file at a known path (here `~/somefile.sif`) then search for another file with the same checksum under `~/.apptainer`:

```
[someuser@node001 [stanage] ~]$ apptainer run docker://alpine:latest whoami
INFO:    Using cached SIF image
someuser
[someuser@node001 [stanage] ~]$ apptainer pull somefile.sif docker://alpine:latest 
INFO:    Using cached SIF image
[someuser@node001 [stanage] ~]$ sha256sum somefile.sif 
09e9acc1061b6683b72d274c41950b99c228040492925c8acb01813e633bde07  somefile.sif
[someuser@node001 [stanage] ~]$ find .apptainer -type f -exec sha256sum '{}' \; | grep 09e9acc1061b6683b72d274c41950b99c228040492925c8acb01813e633bde07
09e9acc1061b6683b72d274c41950b99c228040492925c8acb01813e633bde07  .apptainer/cache/oci-tmp/02bb6f428431fbc2809c5d1b41eab5a68350194fb508869a33cb1af4444c9b11
```

Annoying that `apptainer cache list -v` doesn't tell you more about the location of the cache!
