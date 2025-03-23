MANA: SUSE Linux 13 (Perlmutter)
================================

--------------------------------
Perlmutter Cluster at NERSC/LBNL
--------------------------------

Perlmutter is an HPC cluster running **SUSE Enterprise**.  This document
uses Perlmutter as the model for SUSE Enterprise, to discuss MANA
Checkpointing and Restarting for MPI applications.  However, any given
site may configure SLURM differently.

Perlmutter is a supercomputer at NERSC running SUSE Enterprise.
When it was introduced, it was the #5 supercomputer on the TOP500 list
of June, 2021.  Users should also visit Perlmutter's documentation
`page <https://docs.nersc.gov/getting-started/>`_ for more details.

.. contents:: Contents of this page
   :backlinks: entry
   :local:
   :depth: 2

-------------------------------------
Requesting Resources for Computation
-------------------------------------

There are two methods to request resources on Discovery:

1. **Using SBATCH:**

  * Start by creating a SBATCH script

    .. code:: shell
    
      # Example SBATCH script:
      #!/bin/bash
      #SBATCH -A <account>
      #SBATCH -C cpu
      #SBATCH --qos=debug
      #SBATCH --time=5
      #SBATCH --nodes=2
      #SBATCH --ntasks-per-node=16

      srun -n 32 --cpu-bind=cores -c 16 ./myapp

For SLURM terminology, a _cpu_ is a CPU core, a _node_ is a computer
host, and a _task_ is an MPI process.  The _qos_ (quality of service)
is the type of partition nodes requested.

  * Submit this file as job on cluster

    .. code:: shell
     
      sbatch <sbatch_script>

  * Monitor the output file:
      
    .. code:: shell
    
      tail -f <output_file>

  * To interact with allocated compute nodes:

    .. code:: shell
    
      squeue --me  # squeue -u $USER
      ssh XXX      # where 'XXX' is the allocated node's hostname

2. **Interactive session using salloc:**

  * The **salloc** command is useful for allocating interactive jobs.
    The ``--constraint cpu`` option specifies CPU-only nodes (no GPU).

    .. code:: shell

      salloc --qos interactive --nodes 1 --time 04:00:00 --constraint cpu --account XXX
    
    .. option:: --nodes 1

      Request one node to compute on.
    
    .. option:: --time=04:00:00
    
      Request the node for 4 hours uninterrupted.

    .. option:: --account

      Account name of the project this computation will be charged to.

----------------------------
Compiling MANA on Perlmutter
----------------------------

When  running on the Perlmutter cluster, MANA compilation is recommended
to be performed on a login node.

Steps to compile MANA:

    .. code:: shell
    
      git clone https://github.com/mpickpt/mana
      cd mana
      git submodule init
      git submodule update
      ./configure
      make -j$(nproc)

--------------------------
Testing MANA on Perlmutter
--------------------------

Steps for testing MANA on the Perlmutter cluster:

1. Request one or more compute nodes interactively using salloc:

   ***FIXME: ``salloc`` ... (SEE "salloc", above.)***

2. Open two terminals connected to the same compute node. Compute node
   can be requested using the instructions from above sections. SSH into
   the compute node from a new terminal to get two terminals hooked to same
   compute node. Consider the following points:

   * You can check your hostname to connect via ssh using
     ``squeue --me`` to list all the compute nodes assigned to
     your username.
   * Running ``ssh XXXX`` will connect to your compute node via ssh.
     (Here cXXX is a placeholder for your compute-node name.)

3. Launch a MANA coordinator in Terminal 1:

  .. code:: shell
  
    PATH_TO_MANA/bin/mana_coordinator

  The ``mana_coordinator`` command also supports these command line arguments:

  .. option:: -p, --coord-port PORT_NUM (environment variable DMTCP_COORD_PORT)
  
    Port to listen on (default: 7779)

  .. option:: --port-file filename

    File to write listener port number.
    (Useful with '--port 0', which is used to assign a random port)

  .. option:: --status-file filename

      File to write host, port, pid, etc., info.

  .. option:: --ckptdir (environment variable DMTCP_CHECKPOINT_DIR):

      Directory to store dmtcp_restart_script.sh (default: ./)

  .. option:: --tmpdir (environment variable DMTCP_TMPDIR):

      Directory to store temporary files (default: env var TMPDIR or /tmp)

  .. option:: --write-kv-data:

      Writes key-value store data to a json file in the working directory

  .. option:: --exit-on-last

      Exit automatically when last client disconnects

  .. option:: --kill-after-ckpt

      Kill peer processes of computation after first checkpoint is created

  .. option:: --timeout seconds

      Coordinator exits after <seconds> even if jobs are active
      (Useful during testing to prevent runaway coordinator processes)

  .. option:: --stale-timeout seconds

      Coordinator exits after <seconds> if no active job (default: 8 hrs)
      (Default prevents runaway coord's; Override w/ larger timeout or -1)

  .. option:: --daemon

      Run silently in the background after detaching from the parent process.

  .. option:: -i, --interval (environment variable DMTCP_CHECKPOINT_INTERVAL):

      Time in seconds between automatic checkpoints
      (default: 0, disabled)

  .. option:: --coord-logfile PATH (environment variable DMTCP_COORD_LOG_FILENAME

              Coordinator will dump its logs to the given file

  .. option:: -q, --quiet

      Skip startup msg; Skip NOTE msgs; if given twice, also skip WARNINGs

  .. option:: --help:

      Print this message and exit.

  .. option:: --version:

      Print version information and exit.

4. Launch the MPI process under MANA using srun:

  .. code:: shell
  
    mkdir ckpt_images
    srun -n 2 PATH_TO_MANA/bin/mana_launch.py --ckptdir ckpt_images PATH_TO_MANA/mpi-proxy-split/test/ping_pong.exe

  Use ``mpirun`` instead of ``srun`` if you are using the Open MPI module.

  **NOTE:** Usually, you use ``mana_launch.py`` directly with an executable
  compiled with the local ``mpicc`` command.  For some cases (e.g., MPICH-4.x),
  we have encountered an MPI library that depends on other libraries with
  constructors (e.g., intel, UCX libraries) that gain control before MANA.
  This can interfere with the proper functionig of ``mana_launch.py``.
  If you enounter this,  there are two possible workarounds.

  **NOTE:** For background, a MANA computation uses a split process
  architecture.  Two programs (an upper-half program contains the user MPI
  application, but it uses stub libraries that link MPI calls to an MPI
  library within a lower-half program.  The lower half is a standalone
  MANA-specific MPI application.  AT checkpoint time, only the upper
  half is saved, and at restart time, the lower-half program restores the
  memory of the upper half, and re-binds it to the lower-half MPI library.
  For details, see the original :ref:`MANA paper<mana_paper>`.

  A. For both open and closed source MPI applications, we provide
     an option to use *shadow libraries* for the ``upper half`` of MANA,
     only.  This adds to the library search path a directory of dummy
     libraries to shadow certain libraries related to MPI.  The ``lower
     half`` of MANA uses all of the standard MPI libraries.  The directory
     of shadow libraries is contained in ``PATH_TO_MANA/lib/tmp`` and
     is used ONLY with
     ``mana_launch.py``.

     .. option:: --use-shadowlibs

       Launch MANA and use the shadow libraries in the upper half.

  B. For open source MPI applications, a custom MANA compiler may be used:
     ``PATH_TO_MANA/bin/mpicc_mana``.  (And do not use ``--use-shadowlibs``
     in this case.)

    .. code:: shell
    
       mpicc_mana my_mpi_application.c

5. Create a checkpoint using Terminal 2:

  .. code:: shell
  
    PATH_TO_MANA/bin/mana_status -c

6. Restart from the checkpointed state:

  .. code:: shell
  
    PATH_TO_MANA/bin/mana_restart.py --restartdir ckpt_images

--------------------------------------
Note: three ways to create checkpoints
--------------------------------------
There are three ways to create a checkpoint.

1. Using ``mana_command -c`` as above.

2. Periodic checkpointing with ``-i 60`` (60 seconds). This option
   can be used with either ``mana_coordinator``, ``mana_launch``, or
   ``mana_restart``.

3. In advanced usage, there's a way to request a checkpoint under program control.
