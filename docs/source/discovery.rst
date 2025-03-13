MANA on CentOS 7
================

------------------------------------
Discovery Cluster at Northeastern U.
------------------------------------

Discovery is an HPC cluster running **CentOS 7**.  This document aims to
help users running CentOS-7 to utilize MANA Chekpointing and Restarting
Software with MPI applications.  Since each site can configure Slurm
differently, we use the Discovery cluster at Northeastern U. as our
model, here.

Discovery is a high performance computing
(HPC) resource for the Northeastern University research community.
The Discovery cluster provides access to over 50,000 CPU cores and
over 525 GPUs to all Northeastern faculty and students free of charge.
Compute nodes are connected with either 10 GbE or high data rate
InfiniBand (200 Gbps or 100 Gbps), supporting all types and scales of
computational workloads.  Users should also visit Discovery's information
`page <https://rc.northeastern.edu>`_ for more details.

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

  * The local policy is to do most work, including large compilations
    on a compute node, note the login node.  If you just need a shell
    for compilation, then choose `1` (srun --pty /bin/bash) to obtain
    a shell on a compute node.

3. **Using SRUN:**

  * The **`srun`** command is useful for interactively running jobs, once you
    are on a compute node.  In this example, instead of using `job-assist`,
    we ask for a shell on the command line.

    .. code:: shell

      srun --partition=short --nodes=1 --ntasks=8 --cpus-per-task=1 --time=08:00:00 --mem=8GB --pty /bin/bash
    
    .. option:: --partition=short

      Define type of partition required.
    
    .. option:: --nodes=1

      Request one node to compute on. (Max allowed=2 for short nodes)
    
    .. option:: --ntasks=8

      Number of tasks to run on requested compute resource.
    
    .. option:: --cpus-per-task=1
    
      Inform resource manager that we will run one process per CPU-core.
    
    .. option:: --time=08:00:00
    
      Request the node for 8 hours uninterrupted.
    
    .. option:: --mem=8GB
    
      Requesting 8GB per CPU-core.
    
    .. option:: --pty /bin/bash
    
      Create an interactive shell using `/bin/bash`


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

    .. MANA currently needs gcc-9.  Can we fix it to allow gcc-8?
  
    .. code:: shell
    
      module avail gcc
      module load gcc/9.2.0
      module avail python
      module load python/3.8.1

    And next, choose your preferred MPI.  When in doubt, use
    :code:`module show <modulefile>` to get more information on the
    module.  Here, we see a user switching choices.

    .. code:: shell

      module avail mpi module avail openmpi module load mpich  #
      Accept default: currently mpich/3.3.2 module switch mpich
      mpich/4.0.1-intel2022 module switch mpich openmpi  # Accept default:
      currently openmpi/3.1.2 module list  # Check for compatible gcc,
      python, mpi

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
   
1. Request a compute node interactively:

2. Open two terminals connected to the same compute node. Compute node can be requested using the instructions from above sections. SSH into the compute node from a new terminal to get two terminals hooked to same compute node. Consider the following points:
    
   * Your .ssh dir should be configured to use key-handshake with **`localhost`**. 
    
   * You can check your hostname to connect via ssh using **`squeue --me`** to list all the compute nodes assigned to your username.

   * Running **`ssh cXXXX`** will connect you compute node via a side ssh channel. (here cXXX is a placeholder for your compute-node name)

3. Launch a MANA coordinator in Terminal 1:

  .. code:: shell
  
    /PATH_TO_MANA/bin/mana_coordinator

  The mana_coordinator command also supports these command line arguments:

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

      Directory to store temporary files (default: env var TMDPIR or /tmp)

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
    mpirun -n 2 /PATH_TO_MANA/bin/mana_launch.py --ckptdir ckpt_images /path/to/mana/mpi-proxy-split/test/ping_pong.exe

  **NOTE:** For MPI library versions for Intel compiled with UCX library to support Infiniband, we provide the following two solutions:
      
  A. For Open-Source user MPI-Applciations, we have provided a custom compiler, located at ``/PATH_TO_MANA/bin/mpicc_mana``.

    .. code:: shell
    
       mpicc_mana my_mpi_application.c 

  B. For Closed-Source MPI-Applciations, we provide support of 'shadow library' that creates a lib path of dummy libraries to shadow real Intel and UCX libraries.   
         This creates a shadow library in ``/PATH_TO_MANA/lib/tmp`` and can be used ONLY with ``mana_launch.py``.

   .. option:: --use-shadowlibs

     Launch MANA with support for shadow libraries.
 
5. Signal a checkpoint creation from Terminal 2:

  .. code:: shell
  
    /PATH_TO_MANA/bin/mana_status -c

6. Restart from the checkpointed state:

  .. code:: shell
  
    /PATH_TO_MANA/bin/mana_restart.py --restartdir ckpt_images

--------------------------------------
Note: three ways to create checkpoints
--------------------------------------
There are three ways to create a checkpoint. 

1. Using ``mana_command -c`` as above.

2. Periodical checkpointing with ``-i 60`` (60 seconds). This option can be used with either ``mana_coordinator``, ``mana_launch``, or ``mana_restart``. 

3. In advanced usage, there's a way to request a checkpoint under program control.
