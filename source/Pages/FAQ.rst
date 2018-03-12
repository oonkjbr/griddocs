.. _FAQ:

**************************
Frequently asked questions
**************************

Check out in this page the most commonly asked questions about Grid. If you still have questions, please contact us at helpdesk@surfsara.nl:

.. contents:: 
    :depth: 4  


===============
Getting started
===============

.. _where-to-start:

I 

===============
Using resources 
===============


.. _how-many-cpus:

How many cpu's, nodes does the HT offer?
===========================================


.. _how-many-ch:

How many cpu hours are available?
=================================

The available core hours and storage depend on the funding models. We make tailored agreements to incorporate the user requirements and grant resources based on the applicable funding scheme.


.. _how-much-memory:

What is the average amount of memory available per node?
========================================================

The average memory per node depends on number of cores per node. It is typically 8GB per core, but the nodes vary between 12 and 64 cores per node (48 to 256GB RAM per node).


.. _cpu-time:

How can I calculate the total CPU time I consumed?
==================================================

The total CPU time depends on the amount of cores that your application is using and the wallclock time that the corresponding job takes to finish::

	CPU time = #cores x wallclock(per job) x #jobs	

For example, let's say that a single job takes 12 h to finish on a 4-core machine and we submitted 10,000 of those. The total CPU time spent is::

	CPU time = 4cores x 12h x 10,000 = 480,000 CPU hours ~ 55 CPU years 


.. _cpu-efficiency:

System usage and CPU efficiency
===============================

CPU efficiency is an important factor to detect if the jobs run smoothly on the infrastructure. The CPU efficiency depends on the real CPU usage and the WallClock time for the job to finish::

	CPU efficiency = CPU time / WallClock time

If the CPU was efficiently being used during the job runtime, then a single core job will have efficiency close to 100%. For multicore jobs the efficiency is higher than 100%.


.. _pbs-walltime:

How to run  jobs with wallclock greater than 36 hours on HT?
=========================================================================== 

