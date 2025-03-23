MANA: CentOS 7/Rocky 8
======================

--------------------------------------------
Discovery Cluster at Northeastern University
--------------------------------------------

Discovery is an HPC cluster running **CentOS 7**.  This document uses
Discovery as the model for CentOS 7 and Rocky 8/9, to discuss MANA
Checkpointing and Restarting for MPI applications.  However, any given
site may configure SLURM differently.

Discovery is a high performance computing (HPC) resource for the
Northeastern University research community.  In 2024, the Discovery
cluster provides access to over 50,000 CPU cores and over 525 GPUs to
all Northeastern faculty and students free of charge.  Compute nodes
are connected with either 10 GbE or high data rate InfiniBand (200
Gbps or 100 Gbps), supporting all types and scales of computational
workloads.  Users should also visit `Discovery's information page
<https://rc.northeastern.edu>`_ for more details.

.. contents:: Contents of this page
   :backlinks: entry
   :local:
   :depth: 2

-------------------------------------
Requesting Resources for Computation
-------------------------------------

There are two methods to request resources on Discovery:
batch and interactive.

1. **Using SBATCH:**

  * Start by creating a SBATCH script
    
    .. code:: shell
    
      # Example SBATCH script:
      #!/bin/bash
      #SBATCH -J MyJob           # Job name
      #SBATCH -N 2               # Number of nodes
      #SBATCH -n 16              # Number of tasks
      #SBATCH -o output_%j.txt   # Standard output file
      #SBATCH -e error_%j.txt    # Standard error file
      #SBATCH --mail-user=$USER@northeastern.edu
      #SBATCH --mail-type=ALL
      ./my_program

  * Submit this file as job on cluster
  
    .. code:: shell
     
      sbatch <sbatch_script>

  * Monitor the output file:
      
    .. code:: shell
    
      tail -f <output_file>  

  * To interact with allocated compute nodes:

    .. code:: shell
    
      squeue -u <your_username>
      ssh cXXX  # where 'cXXX' is the allocated node's ID


2. **Interactive Allocation Tool:**

  * Load the job-assist module:
   
    .. code:: shell
     
      module load job-assist

  * Run the interactive tool:
    
    .. code:: shell
     
      job-assist
  
      # Example menu options:
      # SLURM Menu:
      # 1. Default mode (srun --pty /bin/bash)
      # 2. Interactive Mode
      # 3. Batch Mode
      # 4. Exit

  * The policy on Discovery is to run on a compute node, and not the original login node.
    If you just need a shell for compilation, then choose ``1`` (``srun
    --pty /bin/bash``) to obtain a shell on a compute node.

3. **Interactive session using srun:**

  * The **srun** command is useful for interactively running jobs,
    once you are on a compute node.  In this example, instead of using
    ``job-assist``, we ask for a shell on the command line.  Note that
    on Discovery, compute nodes may be shared.  Even if you ask for
    all of the CPU cores (as specified by ``--ntasks``), if you are
    not currently running a job, then the system still may allocate
    another user to the same node.  Further, on Discovery, nodes may
    use either TCP/IP (Ethernet) or InfiniBand.  Optionally, add
    :option:`--constraint=ib` to ``srun`` to request nodes with InfiniBand.
    (Other SLURM sites may name the ``ib`` feature to a different name.)

    .. code:: shell

      srun --partition=short --nodes=1 --ntasks=8 --cpus-per-task=1 --time=08:00:00 --mem=8GB --pty /bin/bash
    
    .. option:: --partition=short

      Define type of partition required.
    
    .. option:: --nodes=1

      Request one node to compute on. (Max allowed=2 for short partitions)
    
    .. option:: --ntasks=8

      Number of tasks (CPU cores) to run on requested compute nodes.
    
    .. option:: --cpus-per-task=1
    
      Inform resource manager that we will run one process per CPU-core.
    
    .. option:: --time=08:00:00
    
      Request the node for 8 hours uninterrupted.
    
    .. option:: --mem=8GB
    
      Requesting 8GB per CPU-core.
    
    .. option:: --constraint=ib
    
      Option specific to Discovery: request InfiniBand nodes
    
    .. option:: --pty /bin/bash
    
      Create an interactive shell using ``/bin/bash```


----------------------------
Compiling MANA on Discovery
----------------------------

When  running on the Discovery cluster, MANA compilation must be performed
on a compute node. Login nodes are restricted from running compilations
or other long commands by the admin.

Steps to compile MANA:

  * Switch to an interactive compute node using the instructions above.
  * Confirm you are on a compute node (hostname should start with 'c'):
  * Set your modules to a reasonable default.  As of early 2025, the
    default is gcc-4.8, python-2.7, and no MPI.  We currently are choosing:

    .. code:: shell
    
      # Check for compatible gcc, python, mpi
      module avail gcc
      module load gcc/8.1.0
      module avail python
      module load python/3.8.1

    And next, choose your preferred MPI.  When in doubt, use
    :code:`module show <modulefile>` to get more information on the
    module.  Here, we see a user switching choices.

    .. code:: shell

      module avail mpi
      module avail mpich
      module avail openmpi # Default is currently openmpi/3.1.2 
      module load mpich # Accept default: currently mpich/3.3.2
      module list

  * Now proceed with installing MANA on Discovery. For more detailed
    instructions, visit the `MANA Home page <https://github.com/mpickpt/mana>`_.

    .. code:: shell

      git clone https://github.com/mpickpt/mana
      cd mana
      git submodule init
      git submodule update
      ./configure
      make -j8

    We use :code:`-j8` because we requested :code:`--ntasks=8` earlier.
    If you are developing software and wish to see internals of MANA,
    choose :code:`./configure --enable-debug` instead.

--------------------------
Testing MANA on Discovery
--------------------------

Steps for testing MANA on the Discovery cluster:

1. Request a compute node interactively.  As before, do:

    .. code:: shell

      srun --partition=short --nodes=1 --ntasks=8 --cpus-per-task=1 --time=08:00:00 --mem=8GB --pty /bin/bash

2. Open two terminals connected to the same compute node. Compute node
   can be requested using the instructions from above sections. SSH into
   the compute node from a new terminal to get two terminals hooked to same
   compute node. Consider the following points:

   * Your .ssh directory should be configured to use a key-handshake with
     **localhost**.
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

4. Launch the MPI process under MANA:

  .. code:: shell
  
    mkdir ckpt_images
    mpirun -n 2 PATH_TO_MANA/bin/mana_launch.py --ckptdir ckpt_images PATH_TO_MANA/mpi-proxy-split/test/ping_pong.exe

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
