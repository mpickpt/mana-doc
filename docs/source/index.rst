.. mana-doc documentation master file, created by
   sphinx-quickstart on Tue Oct  8 15:06:04 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. For the basic syntax of .rst files, see:
   https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html

.. _home:

MANA
==================================================

MANA (MPI-Agnostic Network-Agnostic Checkpointing) is an implementation
of transparent checkpointing for MPI. It is built as a plugin on top
of the `DMTCP checkpointing package <https://github.com/dmtcp/dmtcp>`_.
DMTCP stands for "Distributed MultiThreaded CheckPointing".

.. contents:: Contents of this page
   :backlinks: entry
   :local:
   :depth: 2

----------------------
Installation
----------------------
MANA can be installed using the following instructions:

.. code:: shell

   git clone https://github.com/mpickpt/mana
   cd mana
   git submodule init
   git submodule update
   ./configure
   make -j8

Note that the :command:`git submodule` command will automatically
pull in a special branch of DMTCP built for MANA.  In the future,
DMTCP will evolve so that MANA can use the standard DMTCP.

If you are debugging, use ``./configure --enable-debug``, but MANA
will then be compiled with ``-g3 -O0``, which implies lower performance.

-----------------------------------
Quick Start (single host, no SLURM)
-----------------------------------
This quick start assumes that you do not need to invoke SLURM, for
example on a private PC.  A few HPC sites may also allow you to run using
``mpirun`` on a login node that you ssh to.  But most HPC sites
will require you to login to a compute node, and then use SLURM commands.
For more information, see the section `Running MANA with SLURM`_.


To launch an MPI program with MANA:

.. code:: shell

   PATH_TO_MANA/bin/mana_coordinator
   [mpirun -n <num> ] PATH_TO_MANA/bin/mana_launch.py [mana options] [user_program and args]

To restart a previously checkpointed program:

.. code:: shell
    
   PATH_TO_MANAPATH_TO_MANA/bin/mana_coordinator
   [mpirun -n <num> ] PATH_TO_MANA/bin/mana_restart.py [mana options]

-----------------------------
MANA man page (MANA commands)
-----------------------------
.. program:: mana

**NAME**

.. parsed-literal::

   MANA - MANA is a transparent checkpointing tool for MPI program.

**SYNOPSIS**

.. parsed-literal::

   :program:`mana_coordinator` [OPTIONS]
   :program:`mana_launch.py` [OPTIONS] target_executable
   :program:`mana_restart.py` [OPTIONS]

Options for ``mana_coordinator``:
---------------------------------

.. option:: --verbose

   Print additional information while running.

.. option:: --help

   Print help information and exit.

.. option:: -q, --quiet

   Suppress MANA coordinator's output.

.. option:: -i <seconds>, --interval <seconds>

   Automatically checkpoint after every <seconds> of time.

Options for ``mana_launch.py``:
-------------------------------

.. option:: --verbose

   Print the launching command and options before running the target executable.

.. option:: --help

   Print help information and exit.

.. option:: -h <hostname>, --coord-host <hostname>

   Set the hostname for connecting MANA's coordinator. If not set, MANA will use the local host.

.. option:: -p <port>, --coord-port <port>

   Set the port number for connecting MANA's coordinator. If not set, MANA will use the default port.

.. option:: -i <seconds>, --interval <seconds>

   Automatically checkpoint after every <seconds> of time.

.. option:: --ckptdir <dir>

   Directory in which to save checkpoint images. If not set, it will use the current working directory by default.

.. option:: --ckpt-signal <signum>

   Set the single for checkpoint request. Default signal is SIGUSR2.

.. option:: --with-plugin <path>

   Provide additional plugin for dmtcp.

.. option:: --tmpdir <path>

   Set MANA's tmp directory.

.. option:: --coord-logfile <path>

   Set MANA's coordinator log file path.

.. option:: --gdb

   Launch MANA and the target executable with gdb.

.. option:: --timing

   Print timing information for each stages for checkpoint and restart to stderr.

.. option:: -q, --quiet

   Suppress MANA's output.

.. option:: --use-shadowlibs

   Create dummy library for shadowing original Intel and UCX libraries 

Options for ``mana_restart.py``:
--------------------------------

.. option:: --verbose

   Print the restarting command and options before restarting.

.. option:: --help

   Print help information and exit.

.. option:: -h <hostname>, --coord-host <hostname>

   Set the hostname for connecting MANA's coordinator. If not set, MANA will use the local host.

.. option:: -p <port>, --coord-port <port>

   Set the port number for connecting MANA's coordinator. If not set, MANA will use the default port.

.. option:: -i <seconds>, --interval <seconds>

   Automatically checkpoint after every <seconds> of time.

.. option:: --ckptdir <dir>

   Directory to save checkpoint images. If not set, will use the working directory by default.

.. option:: --restartdir <dir>

   Directory to find checkpoint images. If not set, will use the current working directory by default.

.. option:: --ckpt-signal <signum>

   Set the single for checkpoint request. Default signal is SIGUSR2.

.. option:: --with-plugin <path>

   Provide additional plugin for dmtcp.

.. option:: --tmpdir <path>

   Set MANA's tmp directory.

.. option:: --coord-logfile <path>

   Set MANA's coordinator log file path.

.. option:: --gdb

   Restart MANA and the target executable with gdb.

.. option:: --timing

   Print timing information for each stages for checkpoint and restart to stderr.

.. option:: -q, --quiet

   Suppress MANA's output.

-----------------------
Running MANA with SLURM
-----------------------
SLURM is the most common job scheduler used on HPC clusters.  
This subsection describes the basics elements of using
MANA with SLURM.   After familiarizing yourself with these
basics, please see the section `Running MANA with SLURM: site-specific`_,
and choose the Linux distro most like your own for site-specific details.

On a SLURM site, you will generally first ssh to a *login node*.
Sites may then recommend either to compile on the login node (and possibly
test on the local login node using ``mpirun``), or else to first use SLURM
commands to allocate a *compute node*.  You will need to allocate one
or more compute nodes before running multi-node MPI jobs.

Next, we assume that your system asks you to use ``srun`` to replace
``mpirun`` when running MPI applications with multiple processes. In
this case, you can launch and restart an MPI program with MANA using
the following commands:

.. code:: shell

   PATH_TO_MANA/bin/mana_coordinator
   srun -n <num> PATH_TO_MANA/bin/mana_launch.py [mana options] [user_program and args]
   srun -n <num> PATH_TO_MANA/bin/mana_restart.py [mana options]

MANA will automatically write the hostname and port number of MANA's
coordinator to a file
``.mana.rc`` or ``.mana-slurm-[job id].rc`` and place the file in
user's home directory. Old rc files will be automatically deleted when
using MANA. Users can also manually remove these files after the job
is finished.

When running on multiple nodes, each MPI process will automatically find
the correct hostname and port number of MANA's coordinator using the rc
file. Alternatively, users can manually set the coordinator's hostname
and port number using the ``-h`` and ``-p`` options when launching or
restarting an MPI program with MANA. More information about these two
options can be found in MANA's man page.

Please be aware when deciding the number of process per node and CPU cores
per process that MANA will create an additional checkpoint thread
in each MPI process.

--------------------------------------
Running MANA with SLURM: site-specific
--------------------------------------
Check out the following for site-specific details.  Please choose the
page that is most similar to your site:

1.  :doc:`MANA on CentOS 7 (Discovery at Northeastern University)<discovery>`
2.  :doc:`MANA on SUSE Enterprise Linux 13 (Perlmutter at NERSC/LBNL)<perlmutter>`

If your site is not a good match for any of the above, please send us a
note at github about what are the differences, and we will also document
your site.  You can find the upstream source for this documentation
at https://github.com/mpickpt/mana-doc.

------------------------------------------------
Debugging MPI applications: with or without MANA
------------------------------------------------

Debugging MPI jobs represents a special challenge, whether simply
running ``mpirun`` (without MANA) or running MANA on top of ``mpirun``.
As always, you should go through a hierarchy of validations:

1.  Does your MPI application work with ``mpirun`` (without MANA)?
2.  Does your MPI application work with MANA for a single MPI process?
3.  Does your MPI application work with MANA for many MPI processes
    on a single host?
4.  Does your MPI application work with MANA for two MPI processes,
    with each process on a different host?
5.  Does your MPI application work with MANA for many MPI processes and
    many hosts?

While print statements may help, we concentrate here on using GDB to
attach to a running process.  Therefore, we will first need to ssh
to the compute node on which the MPI process is running.  Ask the IT
staff at your site for the preferred method to ssh to a compute node.
At some sites, you may first need to configure your ssh keys to allow
for using ``ssh`` between nodes on the HPC site without a password.
At other sites, you may need to ssh to an intermediate "data node",
from which you can then ssh to the desired compute node.

We debug using the first environment on which above validation fails.
It is likely that an MPI process has crashed, and so the SLURM job
has been terminated.  If a core dump is created, it may be possible
to identify the offending function.  (See ``gdb-dmtcp-utils`` below,
for interpreting the core dump in GDB.)

Next, we must pause the MPI job before the crash.  To do that, insert
the following code at the beginning of the offending function.

.. code:: C
  int dummmy=1;
  while (dummy);

Now that we have paused the MPI job in an infinite loop, it remains to ssh
to one or more compute nodes.  We can open a separate terminal window for
each process to be debugged.  As noted above, your IT staff can tell you
the preferred method to ssh to a given compute node of your SLURM job.
Once on the compute node, we execute ``gdb -p <PID>`` where PID is the
pid of the the target MPI process.

Once inside GDB and attached to an MPI process, we have found that
whether your are running with native MPI or with MANA, the stack (using
gdb `bt` or `where`) often does not show the functions/line numbers,
even though we compiled the MPI application with `-g`.  In this case,
the solution is to use a GDB utility provided by MANA:

.. code:: C
  (gdb) source PATH_TO_MANA/util/gdb-dmtcp-utils
  (gdb) dmtcp

where `PATH_TO_MANA` is the path to the root directory of the MANA
distribution.  Then try out the extended GDB commands that can restore
the original debug symbols, and also do much more.

Next, after examining the values of any variables, it is time to step
through the execution.  To break out of the infinite loop, we first set
`dummy` to `0`.

.. code:: C
  (gdb) set dummy = 0

Finally, if you wish to step into the functions of MANA itself, then be
sure to configure MANA using `./configure --enable-debug`.

----------------------
Citations
----------------------

.. [mana_paper] \-  "MANA for MPI: MPI-agnostic network-agnostic transparent checkpointing."
   Garg, Rohan, Gregory Price, and Gene Cooperman. Proceedings of the 28th international symposium
   on high-performance parallel and distributed computing. 2019.

.. [optimization] \-  "Enabling Practical Transparent Checkpointing for MPI: A Topological Sort Approach."
   Xu, Yao, and Gene Cooperman. 2024 IEEE International Conference on Cluster Computing (CLUSTER). IEEE, 2024.

-----------
Search page
-----------

* :ref:`search`

.. toctree::
  :hidden:
  :maxdepth: 2

  Home <self>
  discovery   
  perlmutter
