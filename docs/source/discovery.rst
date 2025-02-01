MANA on Discovery
==================================================

MANA (MPI-Agnostic Network-Agnostic Checkpointing) is an implementation
of transparent checkpointing for MPI. Below are instructions to clone, compile,
request resources, and test MANA on the Discovery cluster.

.. contents:: Contents of this page
      :backlinks: entry
   :local:
   :depth: 2

----------------------------
Compiling MANA on Discovery
----------------------------

MANA compilation must be performed on a compute node. Login nodes are restricted from running compilations.

Steps to compile MANA:

.. code:: shell

   # Switch to a compute node:
   srun --pty /bin/bash

   # Confirm you are on a compute node (hostname should start with 'c'):
   hostname

   # Navigate to the MANA directory:
   cd mana

   # Load submodules:
   git submodule init
   git submodule update

   # Configure the make files:
   ./configure             # Normal configuration
   ./configure --enable-debug   # For debug flags (-g3 -O0)

   # Compile the project:
   make -j8

-------------------------------------
Requesting Resources for Computation
-------------------------------------

There are two methods to request resources on Discovery:

1 **Using `sbatch`:**

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

   # Submit the job:
   sbatch <sbatch_script>

   # Monitor the output file:
   tail -f <output_file>

   # To interact with allocated compute nodes:
   squeue -u <your_username>
   ssh cXXX  # Replace 'cXXX' with the allocated node ID

2 **Using the Interactive Allocation Tool:**

.. code:: shell

   # Load the job-assist module:
   module load job-assist

   # Run the interactive tool:
   job-assist

   # Example menu options:
   # SLURM Menu:
   # 1. Default mode (srun --pty /bin/bash)
   # 2. Interactive Mode
   # 3. Batch Mode
   # 4. Exit

---------------------------------
Testing with OSU-MICRO-BENCHMARK
---------------------------------

Follow these steps to test with OSU-MICRO-BENCHMARK:

.. code:: shell

   # Download and extract the benchmark:
   wget <osu_micro_benchmark_url>
   tar -xvf <osu_micro_benchmark.tar.gz>

   # Compile the benchmark (ensure you are on a compute node):
   cd <osu_micro_benchmark_dir>
   ./configure CC=mpicc CXX=mpicxx
   make
   make install

   # Locate and execute the benchmark file:
   cd mpi/pt2pt/standard
   mpirun -n 2 ./osu_multi_lat

--------------------------
Testing MANA on Discovery
--------------------------

Steps for testing MANA on the Discovery cluster:

.. code:: shell

   # 1. Request a compute node interactively:
   srun --pty /bin/bash

   # 2. Open two terminals connected to the same compute node.

   # 3. Launch MANA coordinator in Terminal 1:
   /path/to/mana/bin/mana_coordinator

   # 4. Launch the MPI process under MANA:
   mkdir ckpt_images
   mpirun -n 2 /path/to/mana/bin/mana_launch.py --ckptdir ckpt_images /path/to/mana/mpi-proxy-split/test/ping_pong.mana

   # 5. Signal a checkpoint creation from Terminal 2:
   /path/to/mana/bin/mana_status -c

   # 6. Restart from the checkpointed state:
   /path/to/mana/bin/mana_restart.py --restartdir ckpt_images

----------------------------------
Additional Benchmarks and Remarks
----------------------------------

.. note::

   Additional benchmarks and observations for running MANA on Discovery are coming soon....

Back to :ref:`Home <index>`.
