MANA on Perlmutter
==================================================

------------------
Perlmutter Cluster
-----------------

Perlmutter is a supercomputer at NERSC running SUSE Enterprise.
Users should also visit Perlmutter's documentation `page <https://docs.nersc.gov/getting-started/>`_ for more details.

This document aims to help Perlmutter users as well as other similar Cluster users running **SUSE Enterprise** to utilize MANA Chekpointing and Restarting Software with MPI applications. 

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
  
For Slurm terminology, a `cpu` is a CPU core, a `node` is a computer
host, and a `task` is an MPI process.  The `qos` (quality of service)
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

2. **Using Interactive Job:**

  * **`salloc`** command is useful for allocation of interactive job.

    .. code:: shell

      salloc --qos interactive --nodes 1 --time 04:00:00 --constraint cpu --acount XXX
    
    .. option:: --nodes 1

      Request one node to compute on.
    
    .. option:: --time=04:00:00
    
      Request the node for 4 hours uninterrupted.

    .. option:: --account

      Account name of the project this computation will be charged to.
    
----------------------------
Compiling MANA on Perlmutter
----------------------------

When  running on Perlmutter cluster, MANA compilation is recommended be performed on a login node.

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
   
1. Request a compute node interactively:

2. Open two terminals connected to the same compute node. Compute node can be requested using the instructions from above sections. SSH into the compute node from a new terminal to get two terminals hooked to same compute node. Consider the following points:
    
   * You can check your hostname to connect via ssh using **`squeue --me`** to list all the compute nodes assigned to your username.

   * Running **`ssh XXXX`** will connect you compute node via a side ssh channel.

3. Launch MANA coordinator in Terminal 1:

  .. code:: shell
  
    /PATH_TO_MANA/bin/mana_coordinator

  MANA_Coordinator also supports these command line arguments:

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
    srun -n 2 /PATH_TO_MANA/bin/mana_launch.py --ckptdir ckpt_images /PATH_TO_MANA/mpi-proxy-split/test/ping_pong.exe

  User `mpirun` instead of `srun` if you are using the Open MPI module.

5. Signal a checkpoint creation from Terminal 2:

  .. code:: shell
  
    /PATH_TO_MANA/bin/mana_command -c

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
