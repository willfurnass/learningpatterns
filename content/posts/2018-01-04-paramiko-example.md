+++
title = "Submitting jobs to remote Grid Engine clusters using the Paramiko SSH client"
date = 2018-01-04
tags = ["grid-engine", "python", "ssh"]
featured_image = ""
description = ""
aliases = [
    "/posts-output/2018-01-04-paramiko-example",
]
+++

HPC clusters running the Grid Engine distributed resource manager (job scheduling) software allow jobs to be submitted from a whitelist of 'submit hosts'. 
With the [ShARC and Iceberg](http://docs.hpc.shef.ac.uk/en/latest/) clusters here at the University of Sheffield
the per-cluster lists of permitted submit hosts include all login nodes and all worker nodes;
for ease of management and security, unmanaged hosts (e.g. researchers' workstations) are not added to the list.

If you really want to be able to automate the process of submitting jobs from your own machine then 
one option is to write a script that logs into the cluster via [SSH](https://en.wikipedia.org/wiki/Secure_Shell) then submits a job from there. 
One approach to this is to use [Paramiko](http://www.paramiko.org/), a SSH client and server library for Python.

There are other more sophisticated ways of scripting the execution of commands on remote machines via SSH (e.g. [Fabric](http://docs.fabfile.org), [Ansible](https://www.ansible.com/)) but for simple cases Paramiko may be preferable due to its simplicity and transparency.

Here's a Python script (_only_works with Python 3.x) that demonstrates how to submit a trivial Grid Engine batch job from one's local machine using Paramiko:

[https://gist.github.com/willfurnass/37bec522fea281bd325de421f755c10d](https://gist.github.com/willfurnass/37bec522fea281bd325de421f755c10d)

It works as follows:

1. Create a job submission script on the local filesystem;
1. Start a SSH session using Paramiko and connect to a particular Grid Engine cluster using the hostname of a 'login' node (e.g. `sharc.shef.ac.uk`) and a particular username;
   (I'm using [public-key authentication and a SSH Agent](https://debian-administration.org/article/530/SSH_with_authentication_key_instead_of_password) so 
   no password is needed; embedding passwords in scripts is almost always a Bad Idea);
1. Copy the job submission script to the Grid Engine cluster using [SFTP](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol) (via Paramiko);
1. Run a command on the Grid Engine cluster over SSH to submit the job submission script using Grid Engine's [qsub](http://docs.hpc.shef.ac.uk/en/latest/hpc/scheduler/sge.html) command;
1. Print the standard output, standard error and exit status of that command;
1. The job submission script includes options to ensure that emails are sent when the job starts running and finishes (or aborts).

Here's how you can run this example:

First, ensure you have set up [SSH public-key authentication and a SSH Agent](https://debian-administration.org/article/530/SSH_with_authentication_key_instead_of_password) so 
Paramiko can SSH to a login node of your Grid Engine server without needing a password;

Next, clone the GitHub Gist using [git](https://git-scm.com/):

```
git clone https://gist.github.com/willfurnass/37bec522fea281bd325de421f755c10d paramiko_example
cd paramiko example
```

We now need to make sure the `paramiko` Python package is installed on your local machine.
You could install it globally using your package manager but 
I prefer to install Python packages into isolated 'virtual environments' wherever possible to 
ensure that the requirements of my personal Python projects don't clash with the operating system's Python requirements. 
I've recently started using the awesome [PipEnv](http://pipenv.readthedocs.io/en/latest/) for 
managing different sets of Python packages for different purposes and can highly recommend it. 
First [install PipEnv](http://pipenv.readthedocs.io/en/latest/) then use it to 
install the `paramiko` package into a Python virtual environment that is specific to the current working directory:

```
pipenv install paramiko
```

When I ran the above Paramiko 2.4.0 was installed into my virtual environment.

Finally, edit the `paramiko_test.py` script so it contains _your_ username on the Grid Engine cluster and your email address, then
run the script using:

```
pipenv run python paramiko_test.py
```

When I do this I get:

```
[will@laptop]$ python paramiko_test.py
Standard output:
b'Your job 816720 ("qsub_script_1515071356") has been submitted\n'
Standard error:
b''
Exit status: 0
```

Note that this example is of limited use by itself as the job is executed _asynchronously_; 
by default the `qsub` command queues the job and returns immediately so 
a separate mechanism is needed to determine if and when the job actually executes. 
The exit code printed by the above example (0 here indicates success) only shows 
whether the job submission (and not execution) was successful.

A more robust approach to automating the submission of jobs using Python scripts is 
to use the Python wrapper for the DRMAA API, 
which I previously documented [here](http://docs.hpc.shef.ac.uk/en/latest/hpc/scheduler/drmaa.html), 
or a higher-level workflow manager such as [Ruffus](http://docs.hpc.shef.ac.uk/en/latest/hpc/scheduler/drmaa.html) that depends on that Python package. 
However, note that DRMAA needs to be run from a Grid Engine submit host so you may need to combine Paramiko and DRMAA/Ruffus if 
you want to automate the submission and management of Grid Engine workflows from a machine external to the cluster.
