.. mana-doc documentation master file, created by
   sphinx-quickstart on Tue Oct  8 15:06:04 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

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

----------------------
Quick Start
----------------------
This quick start assumes that you do not need to invoke SLURM, for example
on a private PC.  A few HPC sites may also allow you to run using ``mpirun``
on the login node.  But most HPC sites will require you to login to
a compute node, and then use Slurm commands.
For more information, see the section `Running MANA with SLURM`_.


To launch an MPI program with MANA:

.. code:: shell

   /mana_path/bin/mana_coordinator
   [mpirun -n <num> ] /mana_path/bin/mana_launch.py [mana options] [user_program and args]

To restart a previously checkpointed program:

.. code:: shell
    
   /mana_path/bin/mana_coordinator
   [mpirun -n <num> ] /mana_path/bin/mana_restart.py [mana options]

----------------------
MANA man page
----------------------
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

   Supress MANA coordinator's output.

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

   Supress MANA's output.

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

   Supress MANA's output.

-----------------------
Running MANA with SLURM
-----------------------
SLURM is the most common job scheduler on HPC clusters.  Generally,
you will login to a ``login node``.  Sites may recommend either to
compile on the login node (and possibly test on the local node using
``mpirun``), or else to first use SLURM commands to allocate a compute node.
In general, you will need to allocate a compute node before running
multi-node MPI jobs.

For debugging, when the job is running, many HPC sites allow
you to interactively login to one of the compute nodes running your job,
where you can do things like attaching with ``gdb``.  At some sites,
you may first need to configure your ssh keys to allow for using ``ssh``
between nodes on the HPC site without a password.

Next, we assume that your system asks you to use ``srun`` to replace
``mpirun`` when running MPI applications with multiple processes. In
this case, you can launch and restart an MPI program with MANA using
the following commands:

.. code:: shell

   /mana_path/bin/mana_coordinator
   srun -n <num> /mana_path/bin/mana_launch.py [mana options] [user_program and args]
   srun -n <num> /mana_path/bin/mana_restart.py [mana options]

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

Please be aware when deciding the number of process per node and CPU
binding strategy that MANA will create an additional checkpoint thread
in each MPI process.


---------------------------------------------
Running MANA on a Specific Linux/Slurm distro
---------------------------------------------
Check out the following for site-specific details.  Please choose the page that is most similar to your site:

1.  :doc:`MANA on CentOS 7 (Discovery at Northeastern University)<discovery>`
2.  :doc:`MANA on SUSE Enterprise Linux 13 (Perlmutter at NERSC/LBNL)<perlmutter>`

If your site is not a good match for any of the above, please send us a
note at github about what are the differences, and we will also document
your site.  You can find the upstream source for this documentation
at https://github.com/mpickpt/mana-doc.

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
