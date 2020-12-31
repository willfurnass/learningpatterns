+++
title = "When will my Grid Engine job run?"
date = 2018-01-18
tags = ["grid-engine"]
featured_image = ""
description = ""
aliases = [
    "/posts-output/2018-01-18-sge-priorities",
]
+++


After submitting a job to a Grid Engine (GE) cluster such as [ShARC](http://docs.hpc.shef.ac.uk), GE iterates through all job queues instances on all nodes (in a particular order) to see if there are sufficient free resources to start your job.  If so (and GE isn't holding resources back for a Resource Reservation) then your job will start running.

If not, then your job is added to the list of pending jobs.  Each pending job has an associated dynamically-calculated priority, which the GE scheduler uses to determine the order in which consider pending jobs. You can see the priority for the running and pending jobs of all users with:

```
qstat -u '*' | less
```

It might be useful to prefix these lines with a line number so you can determine roughly how many jobs might have to run before your will ('roughly' as this numbering does not account jobs having the same priority and for 'Resource Reservations' (more on those later)).

```
qstat -u '*' | nl -v -2 | less
```

## How are job priorities calculated?

So what influences that priority?  How is it calculated?

Well, at the highest level it is a function of three components:

```
priority = urgency_weighting        x normalised(urgency) + 
           ticket_weighting         x normalised(tickets) + 
           posix_priority_weighting x normalised(posix_priority)
```

I'll explain each of those three components in turn, focussing primarily on what's important on ShARC to simplify matters.  
NB the names above differ ever so slightly from those used in the Grid Engine documentation.

## Urgency 

The *urgency* of a job is itself a function of three things:

* **Resources requested**: The idea here is that the more resources you request, the more urgent your job is considered to be.  The administrator can assign weightings per type of resource (see the last column of the output of `qconf -sc`).  On ShARC only the requested number of slots (CPU cores) is given a >0 weighting;
* **Waiting time**: i.e. how many seconds since the job was submitted.  This is multiplied by a weight (which is >0 on ShARC);
* **Deadline**: some users are able to submit jobs that must complete before a particular deadline.  For these jobs this priority component increases as one approaches the deadline.


## Tickets (fair share policy)

Grid Engine tracks resources used by different users, GE Projects and Departments over time and can use this information to ensure fair access to resources and specifically, in the case of ShARC, temporarily reduce priorities for those who have recently used lots of resources.

On ShARC, this penalisation for resource usage is based on the CPU and memory resources *used* (not requested).  We could also penalise for IO activity but we don't:

```
$ qconf -ssconf | grep usage_weight_list
usage_weight_list                 cpu=0.500000,mem=0.500000,io=0.000000
```

We also don't penalise for GPU usage as there's no mechanism for this in GE.  This is unfortunate as we'd *really* like to be able to provide fairer access to GPUs.

**TODO: COMPLETE**


## 'POSIX' Priority

The third main component in the overall priority calculations.  ShARC users: this is not used on ShARC so feel free to skip over this section.  

Users can explicitly specific a 'POSIX' priority value when they submit a job by using the `-p SOMENUMBER` argument to `qsub`/`qrsh`/`qsh`.
The default is `0`.  Unprivileged users can specify a value in the range `0` down to `-1023` (to submit lower-priority jobs), but administrators can specify values in the range `-1023` to `1024`.

On ShARC the POSIX priority *weighting* is set to zero, so effectively values specified with `-p` have no effect on job priority.

## Conclusion

**TODO**

---
For (much) more info run the following on ShARC to see relevant [man](https://en.wikipedia.org/wiki/Man_page) pages:

* `man 5 sge_priority`
* `man 5 share_tree`

## SCRAPPY NOTES BELOW

* share-based: share tree policy assigns priorities based on historical usage with a specified lifetime, 
* functional: functional policy only takes current usage into account.  Not used on ShARC
* override

```
urg =  rrcontr + wtcontr + dlcontr

rrcontr = Sum over all(hrr)
hrr = rurg * assumed_slot_allocation * request (numeric complexes)
hrr = rurg (string complexes)

wtcontr = waiting_time * weight_waiting_time

dlcontr = weight_deadline / free_time
or 0 for non-deadline jobs
urg =  rrcontr + wtcontr + dlcontr

rrcontr = Sum over all(hrr)
hrr = rurg * assumed_slot_allocation * request (numeric complexes)
hrr = rurg (string complexes)
```




It's influenced by a number of factors:

 - Who you are;
 - Which Project you submitted your job under (or the default project that you submit jobs under, which is `SHEFFIELD` on ShARC);
 - The resources requested by the job;
 - When the job was submitted;
 - DEADLINE
 - POSIX PRIORITY

Controlled by policy.
Three parts, weightings controlled by 
weight_priority (POSIX) - zero on on ShARC
weight_ticket (1) and 
weight_urgency (0.5).


```
npprior = normalized(ppri)
nurg    = normalized(urg)
ntckts  = normalized(tckts)

prio = weight_priority * npprio +
       weight_urgency  * nurg +
       weight_ticket   * ntckts

urg =  rrcontr + wtcontr + dlcontr

rrcontr = Sum over all(hrr)
hrr = rurg * assumed_slot_allocation * request (numeric complexes)
hrr = rurg (string complexes)

wtcontr = waiting_time * weight_waiting_time

dlcontr = weight_deadline / free_time
or 0 for non-deadline jobs


tckts = ftckt + otckt + stckt
```

The ticket policies provide a broad range of means for influencing both job dispatch
and  runtime  priorities  on  a  per  job, per user, per project, and per department
basis.  The qstat(1) -ext option displays ticket information for jobs.

TICKETS

* share-based: share tree policy assigns priorities based on historical usage with a specified lifetime, 
* functional: functional policy only takes current usage into account.  Not used on ShARC
* override

cs1wf:
200000 share tree tickets
200000 total share tree tickets
0 functional tickets
0 total functional tickets
0 override tickets

share override tickets: on - irreleavnt
share functional shares: on - irreleavnt

Why won't my job run even though there are sufficient free resources?

Possibly due to resource reservation and backfilling



