+++
title = "Running job arrays on HPC using Singularity and a Docker image"
date = 2018-04-05
tags = ["job-arrays", "singularity", "docker", "grid-engine"]
featured_image = ""
description = ""
+++

I thought I'd share a little success story.

A researcher recently approached me to say he'd been having difficulty 
getting his bioinformatics workflow (based on [this][gatk-workflow]) working on the University of Sheffield's [ShARC][sharc] HPC cluster and 
had an urgent need to get it running to meet a pending deadline.
He's primarily using the [GATK][gatk] 'genome analysis toolkit'. 

He appealed for help on the GATK forum, 
members of which suggested that the issue may be due to the way in which GATK was installed. 
They recommended that he try running his workflow using the Docker image produced by the authors of GATK (
[`broadinstitute/gatk:4.0.2.0`][gatk-dockerhub]), 
as within that image GATK plus dependencies had been installed and configured in a well-defined, entirely reproducible way.

The researcher then asked if Docker could be installed on ShARC, 
but there are some [fairly sound technical reasons][kurtzer-interview] why 
system administrators would not want to do that on a traditional multi-user HPC system.

However, there is a similar tool called [Singularity][singularity], 
mentioned in my [previous post](/posts-output/2018-01-30-abaqus-singularity.md)
that was designed from the ground up to be well-suited to HPC environments.  
This is already installed on ShARC and can create its own images from Docker images, 
often without the need for manual intervention. 

## Creating Singularity containers using Docker images

There are two ways to do it:

1.  Convert the specified Docker image to a Singularity image on-the-fly each time you run Singularity. 
    If the conversion process takes a few minutes (as the image is large) then this may not be ideal;
2.  Explicitly convert the Docker image to a Singularity image file that is saved in a known location. 
    This can then be used to near-instaneously create new Singularity containers.

The GATK Docker image is several GB in size so we opted for #2.

To perform this explicit image format conversion, we first started an interactive session on ShARC...

    $ qrshx -l rmem=4G

...requesting more than the default 2GB RAM as we need enough for the image conversion process.

Next, we do the (one-off) image conversion itself. 

    $ export SINGULARITY_CACHEDIR="/fastdata/$USER/singularity-cachedir"
    $ mkdir -p $SINGULARITY_CACHEDIR

    # Create a Singularity container from a Docker container
    $ singularity pull docker://broadinstitute/gatk:4.0.2.0 

    # Go make a cup of tea

This takes a couple of minutes. 

NB I've heard that converting Docker images to Singularity images 
without being the root user *may* cause issues 
if the Docker container uses certain Docker features. 
This wasn't a problem in this case, but if it was the solution would be to

1. install Singularity on your own Linux desktop then
1. run the `singularity pull` step on your own machine using `sudo`.

This would create a `gatk-4.0.2.0.simg` file that you can copy to ShARC.

## Image storage

Why did we need to define `SINGULARITY_CACHEDIR`? 

Well, Singularity by default caches images created from Docker images in your home directory 
(regardless of which conversion process, #1 or #2 above, you choose).
This is sensible on personal machines but can be problematic on shared machines where 
you might have a fairly restrictive quota for your home directory
(home directory quotas on ShARC are 10GB) and it's all too easy for Singularity to gobble up your quota.

To work around this you can tell Singularity to stick the result of image conversions elsewhere. 
Here we define `SINGULARITY_CACHEDIR` to be directory on 
a [Lustre filesystem where there are no quotas][sharc-filestore].
Singularity image files explicitly created from Docker images (not from on-the-fly image conversion) will be stored there.

## Running applications in your container

If your image built successfully you should now be able start a shell in a GATK Singularity container:

    $ singularity run $SINGULARITY_CACHEDIR/gatk-4.0.2.0.simg
    groups: cannot find name for group ID 20000 
  
(NB this error is irrelevant and relates to the [nameless groups created by Grid Engine for job management][sge-job-groups]).

We're now running a shell in our Singularity container, which contains GATK! 
Just type `gatk` to run GATK within the container.

    (gatk) te1st@sharc-node123:/home/te1st$ gatk

    Usage template for all tools (uses --spark-runner LOCAL when used with a Spark tool)
       gatk AnyTool toolArgs
    ...

To exit the container, just type `exit` to quit the shell in the container.

More info on using Singularity on ShARC can be found [here][sharc-singularity].

## Running Singularity within (an array) of batch jobs

The next problem was that we needed to repeat a (two-stage) analysis using GATK for 24 chromosome names: 
1...22, X and Y.

When using a cluster running [Grid Engine][sge] (a *distributed resource manager* / job scheduler) such as ShARC,
the most efficient way to run a set of near identical, independent, non-trivial computational tasks is often to 
submit them as a [job array][sharc-job-array].

Here we effectively submitted a Grid Engine batch job submission script that 
results in 24 distinct tasks being submitted to the cluster's job queues.
The only thing that differs between the tasks is the value of the environment variable
`SGE_TASK_ID`, which we use to determine the chromosome name we want to use for that task.
These tasks can be run independently, allowing some/all to be run in parallel if sufficient computational resources are available.

This is the Grid Engine submission script we used:

```bash
#!/bin/bash
#$ -P rse
#$ -q rse.q
#$ -l mem=12G
#$ -t 1-24
#$ -l h_rt=24:00:00
#$ -N GenomicsDBIimport
#$ -M some.user@sheffield.ac.uk
#$ -m bea

# A Bash array of 24 chromosome names
CHROMOSOME_NAMES=($(seq 1 22) X Y)

# Get the chromosome name for this task,
# using SGE_TASK_ID as an index into the CHROMOSOME_NAMES array
c=${CHROMOSOME_NAMES[$(($SGE_TASK_ID - 1))]}

# Specify where our Singularity image is located
export SINGULARITY_CACHEDIR="/fastdata/$USER/singularity-cachedir"

# Step 1 of the GATK workflow for this chromosome
singularity exec $SINGULARITY_CACHEDIR/gatk-4.0.2.0.simg gatk GenomicsDBImport \
   -R /fastdata/$USER/reference/hs37d5.fa \
   --variant /fastdata/te1st/WGS_MShef7_iPS/output/24811_1_1.bam.g.vcf \
   --variant /fastdata/te1st$USER/WGS_MShef7_iPS/output/24150_1_1.bam.g.vcf \
   --variant /fastdata/te1st/WGS_MShef7_iPS/output/24144_2_1.bam.g.vcf \
   --variant /fastdata/te1st/WGS_MShef7_iPS/output/24712_6_1.bam.g.vcf \
   --variant /fastdata/te1st/WGS_MShef7_iPS/output/24811_2_1.bam.g.vcf \
   --genomicsdb-workspace-path /fastdata/te1st/WGS_MShef7_iPS/output/wt_mshef7_database$c \
   --intervals $c \
   --java-options -DGATK_STACKTRACE_ON_USER_EXCEPTION=true

# Step 2 of the GATK workflow for this chromosome
singularity exec $SINGULARITY_CACHEDIR/gatk-4.0.2.0.simg gatk GenotypeGVCFs \
   -R /fastdata/te1st/reference/hs37d5.fa \
   -V gendb:///fastdata/te1st/WGS_MShef7_iPS/output/wt_mshef7_database$c \
   -O /fastdata/te1st/WGS_MShef7_iPS/output/wt_mshef7_raw_variants_jointcalls_chr$c.vcf \
   --java-options -DGATK_STACKTRACE_ON_USER_EXCEPTION=true
```

Note that here we used `singularity exec` and not `singularity run`: 
this is because we don't want to start an interative shell within our container(s) then
start GATK from there as we want to submit a batch job and can't do anything interactively!
`singularity exec` allows you to run a specific command inside your container.

## Result!

GATK behaved as required, in part due to using a software environment (Docker container) configured by the GATK developers.  Packaging up the analysis as an array job resulted in all 24 tasks executing in a timely manner and the researcher was happy!

[sharc]: http://docs.hpc.shef.ac.uk/en/latest/sharc/
[gatk]: https://software.broadinstitute.org/gatk/
[gatk-dockerhub]: https://hub.docker.com/r/broadinstitute/gatk/
[singularity]: https://singularity.lbl.gov/
[kurtzer-interview]: http://www.admin-magazine.com/HPC/Articles/Interview-with-Gregory-Kurtzer-Developer-of-Singularity
[sharc-filestore]: http://docs.hpc.shef.ac.uk/en/latest/hpc/filestore.html
[sge-job-groups]: http://docs.hpc.shef.ac.uk/en/latest/troubleshooting.html#warning-about-groups-cannot-find-name-for-group-id-xxxxx
[sharc-singularity]: http://docs.hpc.shef.ac.uk/en/latest/sharc/software/apps/singularity.html
[soge]: https://arc.liv.ac.uk/trac/SGE
[sharc-job-array]: http://docs.hpc.shef.ac.uk/en/latest/parallel/JobArray.html
[gatk-workflow]: https://software.broadinstitute.org/gatk/best-practices/workflow?id=11145
