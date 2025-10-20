
******
Vapor
******

.. contents::

System Overview
===============

.. note::
    Access to this system is only available through special projects in NCCS and NCRC with explicit approval. 
    This cluster is not intended for general-use purposes.


Vapor is an Azure-based cluster provides experimental high-performance computing resources for the
NCRC. 

The cluster is configured as follows:

- **Login Node:** 1 node 
- **Compute Nodes:** 8 nodes 
- **Interconnect:** InfiniBand 
- **Processor:** AMD EPYC 7V73X 64-Core Processor
- **Cores per Node:** 120 cores 
- **GPUs:** None 

File Systems
------------

The cluster offers two types of filesystems for storage and computation:

1. **NFS Home Directories:** 
   - Accessible at ``/ccs/home/<username>``

2. **Lustre Filesystem:** 
   - A high-performance Lustre system with 1 TB total available storage, mounted at ``/lustre/OLCFLustre``

Storage Areas Overview
----------------------

The Lustre filesystem is organized into the following storage areas for project-specific work:

+---------------------+-----------------------------------------------------+---------+-------------+------------------+
| Area                | Path                                                | Type    | Permissions | On Compute Nodes |
+=====================+=====================================================+=========+=============+==================+
| Member Work         | ``/lustre/OLCFLustre/[projid]/scratch/[userid]``    | Lustre  | 700         | Yes              |
+---------------------+-----------------------------------------------------+---------+-------------+------------------+
| Project Work        | ``/lustre/OLCFLustre/[projid]/proj-shared``         | Lustre  | 770         | Yes              |
+---------------------+-----------------------------------------------------+---------+-------------+------------------+
| World Work          | ``/lustre/OLCFLustre/[projid]/world-shared``        | Lustre  | 775         | Yes              |
+---------------------+-----------------------------------------------------+---------+-------------+------------------+



Logging In to Vapor 
--------------------

To log in to Vapor, use your OLCF username and passcode:

First ssh to hub and from there ssh to the Vapor login node.


.. code-block:: 

    $ ssh <username>@hub.ccs.ornl.gov
    $ ssh <username>@login1.vapor.olcf.ornl.gov


Programming Environment
=======================

Frontier users are provided with many pre-installed software packages and scientific libraries. To facilitate this, environment management tools are used to handle necessary changes to the shell.

Environment Modules (Lmod)
--------------------------

Environment modules are provided through `Lmod <https://lmod.readthedocs.io/en/latest/>`__, a Lua-based module system for dynamically altering shell environments. By managing changes to the shell’s environment variables (such as ``PATH``, ``LD_LIBRARY_PATH``, and ``PKG_CONFIG_PATH``), Lmod allows you to alter the software available in your shell environment without the risk of creating package and version combinations that cannot coexist in a single environment.

General Usage
^^^^^^^^^^^^^

The interface to Lmod is provided by the ``module`` command:

+------------------------------------+-------------------------------------------------------------------------+
| Command                            | Description                                                             |
+====================================+=========================================================================+
| ``module -t list``                 | Shows a terse list of the currently loaded modules                      |
+------------------------------------+-------------------------------------------------------------------------+
| ``module avail``                   | Shows a table of the currently available modules                        |
+------------------------------------+-------------------------------------------------------------------------+
| ``module help <modulename>``       | Shows help information about ``<modulename>``                           |
+------------------------------------+-------------------------------------------------------------------------+
| ``module show <modulename>``       | Shows the environment changes made by the ``<modulename>`` modulefile   |
+------------------------------------+-------------------------------------------------------------------------+
| ``module spider <string>``         | Searches all possible modules according to ``<string>``                 |
+------------------------------------+-------------------------------------------------------------------------+
| ``module load <modulename> [...]`` | Loads the given ``<modulename>``\(s) into the current environment       |
+------------------------------------+-------------------------------------------------------------------------+
| ``module use <path>``              | Adds ``<path>`` to the modulefile search cache and ``MODULESPATH``      |
+------------------------------------+-------------------------------------------------------------------------+
| ``module unuse <path>``            | Removes ``<path>`` from the modulefile search cache and ``MODULESPATH`` |
+------------------------------------+-------------------------------------------------------------------------+
| ``module purge``                   | Unloads all modules                                                     |
+------------------------------------+-------------------------------------------------------------------------+
| ``module reset``                   | Resets loaded modules to system defaults                                |
+------------------------------------+-------------------------------------------------------------------------+
| ``module update``                  | Reloads all currently loaded modules                                    |
+------------------------------------+-------------------------------------------------------------------------+

Searching for Modules
^^^^^^^^^^^^^^^^^^^^^

Modules with dependencies are only available when the underlying dependencies, such as compiler families, are loaded. Thus, ``module avail`` will only display modules that are compatible with the current state of the environment. To search the entire hierarchy across all possible dependencies, the ``spider`` sub-command can be used as summarized in the following table.

+------------------------------------------+--------------------------------------------------------------------------------------+
| Command                                  | Description                                                                          |
+==========================================+======================================================================================+
| ``module spider``                        | Shows the entire possible graph of modules                                           |
+------------------------------------------+--------------------------------------------------------------------------------------+
| ``module spider <modulename>``           | Searches for modules named ``<modulename>`` in the graph of possible modules         |
+------------------------------------------+--------------------------------------------------------------------------------------+
| ``module spider <modulename>/<version>`` | Searches for a specific version of ``<modulename>`` in the graph of possible modules |
+------------------------------------------+--------------------------------------------------------------------------------------+
| ``module spider <string>``               | Searches for modulefiles containing ``<string>``                                     |
+------------------------------------------+--------------------------------------------------------------------------------------+

Compilers
---------

AMD, GCC, Intel, and LLVM compilers are provided through modules. There is also the system version
of GCC available in ``/usr/bin``. The below table lists details. 


+--------+----------------+----------+--------------+
| Vendor | Compiler       | Language | Compiler     |
+========+================+==========+==============+
| AMD    | aocc           | C        | ``clang``    |
|        |                +----------+--------------+
|        |                | C++      | ``clang++``  |
|        |                +----------+--------------+
|        |                | Fortran  | ``flang``    |
+--------+----------------+----------+--------------+
| Intel  | oneapi         | C        | ``icx``      |
|        |                +----------+--------------+
|        |                | C++      | ``icpx``     |
|        |                +----------+--------------+
|        |                | Fortran  | ``ifx``      |
+--------+----------------+----------+--------------+
| LLVM   | llvm           | C        | ``clang``    |
|        |                +----------+--------------+
|        |                | C++      | ``clang++``  |
|        |                +----------+--------------+
|        |                | Fortran  | ``flang``    |
+--------+----------------+----------+--------------+
| GCC    | gcc            | C        | ``gcc``      |
|        |                +----------+--------------+
|        |                | C++      | ``g++``      |
|        |                +----------+--------------+
|        |                | Fortran  | ``gfortran`` |
+--------+----------------+----------+--------------+

MPI
---

Both MPICH and OpenMPI modules are available. But MPICH is recommended and loaded by default. Use
``mpicc``, ``mpicxx``, ``mpifort`` compiler wrappers for compiling for C, C++, Fortran with MPI. The
compiler wrapper will use the compiler from the currently loaded compiler module.


Running Jobs
============

Computational work on Vapor is performed by *jobs*. Jobs typically consist of several componenets:

-  A batch submission script 
-  A binary executable
-  A set of input files for the executable
-  A set of output files created by the executable

In general, the process for running a job is to:

#. Prepare executables and input files.
#. Write a batch script.
#. Submit the batch script to the batch scheduler.
#. Optionally monitor the job before and during execution.

The following sections describe in detail how to create, submit, and manage jobs for execution on Frontier. Frontier uses SchedMD's Slurm Workload Manager as the batch scheduling system.


Login vs Compute Nodes
----------------------

Recall from the System Overview that Frontier contains two node types: Login and Compute. When you connect to the system, you are placed on a *login* node. Login nodes are used for tasks such as code editing, compiling, etc. They are shared among all users of the system, so it is not appropriate to run tasks that are long/computationally intensive on login nodes. Users should also limit the number of simultaneous tasks on login nodes (e.g., concurrent tar commands, parallel make 

Compute nodes are the appropriate place for long-running, computationally-intensive tasks. When you start a batch job, your batch script (or interactive shell for batch-interactive jobs) runs on one of your allocated compute nodes.


Running Jobs
============

This section describes how to run programs on the Vapor compute nodes,
including a brief overview of Slurm and also how to map processes and threads
to CPU cores and GPUs.

Login vs Compute Nodes
----------------------

Vapor contains two node types: Login and Compute. When you connect to the system, you are placed on a *login* node. Login nodes are used for tasks such as code editing, compiling, etc. They are shared among all users of the system, so it is not appropriate to run tasks that are long/computationally intensive on login nodes. Users should also limit the number of simultaneous tasks on login nodes (e.g., concurrent tar commands, parallel make 

Compute nodes are the appropriate place for long-running, computationally-intensive tasks. When you start a batch job, your batch script (or interactive shell for batch-interactive jobs) runs on one of your allocated compute nodes.

Slurm Workload Manager
----------------------

`Slurm <https://slurm.schedmd.com/>`__ is the workload manager used to interact
with the compute nodes on Vapor. In the following subsections, the most
commonly used Slurm commands for submitting, running, and monitoring jobs will
be covered, but users are encouraged to visit the official documentation and
man pages for more information.

Batch Scheduler and Job Launcher
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Slurm provides 3 ways of submitting and launching jobs on Vapor's compute
nodes: batch  scripts, interactive, and single-command. The Slurm commands
associated with these methods are shown in the table below and examples of
their use can be found in the related subsections.

+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Command    |  Description                                                                                                                                                                 |
+============+==============================================================================================================================================================================+
| ``sbatch`` | | Used to submit a batch script to allocate a Slurm job allocation. The script contains options preceded with ``#SBATCH``.                                                   |
|            | | (see Batch Scripts section below)                                                                                                                                          |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``salloc`` | | Used to allocate an interactive Slurm job allocation, where one or more job steps (i.e., ``srun`` commands) can then be launched on the allocated resources (i.e., nodes). |
|            | | (see Interactive Jobs section below)                                                                                                                                       |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| ``srun``   | | Used to run a parallel job (job step) on the resources allocated with sbatch or ``salloc``.                                                                                |
|            | | If necessary, srun will first create a resource allocation in which to run the parallel job(s).                                                                            |
|            | | (see Single Command section below)                                                                                                                                         |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+ 

Batch Scripts
"""""""""""""

A batch script can be used to submit a job to run on the compute nodes at a
later time. In this case, stdout and stderr will be written to a file(s) that
can be opened after the job completes. Here is an example of a simple batch
script:

.. code-block:: bash

   #!/bin/bash
   #SBATCH -A <project_id>
   #SBATCH -J <job_name>
   #SBATCH -o %x-%j.out
   #SBATCH -t 00:05:00
   #SBATCH -p <partition> 
   #SBATCH -N 2
 
   srun -n4 --ntasks-per-node=2 ./a.out 

The Slurm submission options are preceded by ``#SBATCH``, making them appear as
comments to a shell (since comments begin with ``#``). Slurm will look for
submission options from the first line through the first non-comment line.
Options encountered after the first non-comment line will not be read by Slurm.
In the example script, the lines are:

+------+-------------------------------------------------------------------------------+
| Line | Description                                                                   |
+======+===============================================================================+ 
| 1    | [Optional] shell interpreter line                                             |
+------+-------------------------------------------------------------------------------+ 
| 2    | OLCF project to charge                                                        |
+------+-------------------------------------------------------------------------------+ 
| 3    | Job name                                                                      |
+------+-------------------------------------------------------------------------------+ 
| 4    | stdout file name ( ``%x`` represents job name, ``%j`` represents job id)      |
+------+-------------------------------------------------------------------------------+ 
| 5    | Walltime requested (``HH:MM:SS``)                                             |
+------+-------------------------------------------------------------------------------+ 
| 6    | Batch queue                                                                   |
+------+-------------------------------------------------------------------------------+ 
| 7    | Number of compute nodes requested                                             |
+------+-------------------------------------------------------------------------------+ 
| 8    | Blank line                                                                    |
+------+-------------------------------------------------------------------------------+
| 9    | ``srun`` command to launch parallel job (requesting 4 processes - 2 per node) | 
+------+-------------------------------------------------------------------------------+

.. _interactive:

Interactive Jobs
""""""""""""""""

To request an interactive job where multiple job steps (i.e., multiple srun
commands) can be launched on the allocated compute node(s), the ``salloc``
command can be used:

.. code-block:: bash
   
   $ salloc -A <project_id> -J <job_name> -t 00:05:00 -p <partition> -N 2
   salloc: Granted job allocation 313
   salloc: Waiting for resource configuration
   salloc: Nodes vapor[01-02] are ready for job

   $ srun -n 4 --ntasks-per-node=2 ./a.out
   <output printed to terminal>
 
   $ srun -n 2 --ntasks-per-node=1 ./a.out
   <output printed to terminal>

Here, ``salloc`` is used to request an allocation of compute nodes for
5 minutes. Once the resources become available, the user is granted access to
the compute nodes (``vapor01`` and ``vapor02`` in this case) and can launch job
steps on them using srun. 

.. _single-command:

Single Command (non-interactive)
""""""""""""""""""""""""""""""""

.. code-block:: bash

   $ srun -A <project_id> -t 00:05:00 -p <partition> -N 2 -n 4 --ntasks-per-node=2 ./a.out
   <output printed to terminal>

The job name and output options have been removed since stdout/stderr are
typically desired in the terminal window in this usage mode.

Common Slurm Submission Options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The table below summarizes commonly-used Slurm job submission options:

+--------------------------+--------------------------------+
| Flag                     | Description                    |
+==========================+================================+
| ``A <project_id>``       | Project ID to charge           |
+--------------------------+--------------------------------+
| ``-J <job_name>``        | Name of job                    |
+--------------------------+--------------------------------+
| ``-p <partition>``       | Partition / batch queue        |
+--------------------------+--------------------------------+
| ``-t <time>``            | Wall clock time <``HH:MM:SS``> |
+--------------------------+--------------------------------+
| ``-N <number_of_nodes>`` | Number of compute nodes        |
+--------------------------+--------------------------------+
| ``-o <file_name>``       | Standard output file name      |
+--------------------------+--------------------------------+
| ``-e <file_name>``       | Standard error file name       |
+--------------------------+--------------------------------+

For more information about these and/or other options, please see the
``sbatch`` man page.

Other Common Slurm Commands
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The table below summarizes commonly-used Slurm commands:

+--------------+---------------------------------------------------------------------------------------------------------------------------------+
| Command      |  Description                                                                                                                    |
+==============+=================================================================================================================================+
| ``sinfo``    | | Used to view partition and node information.                                                                                  |
|              | | E.g., to view user-defined details about the batch partition:                                                                 |
|              | | ``sinfo -p partition -o "%15N %10D %10P %10a %10c %10z"``                                                                     | 
+--------------+---------------------------------------------------------------------------------------------------------------------------------+
| ``squeue``   | | Used to view job and job step information for jobs in the scheduling queue.                                                   |
|              | | E.g., to see all jobs from a specific user:                                                                                   |
|              | | ``squeue -l -u <user_id>``                                                                                                    |
+--------------+---------------------------------------------------------------------------------------------------------------------------------+
| ``sacct``    | | Used to view accounting data for jobs and job steps in the job accounting log (currently in the queue or recently completed). |
|              | | E.g., to see a list of specified information about all jobs submitted/run by a users since 1 PM on October 10, 2025           |
|              | | ``sacct -u <username> -S 2025-10-04T13:00:00 -o "jobid%5,jobname%25,user%15,nodelist%20" -X``                                 |
+--------------+---------------------------------------------------------------------------------------------------------------------------------+
| ``scancel``  | | Used to signal or cancel jobs or job steps.                                                                                   |
|              | | E.g., to cancel a job:                                                                                                        |
|              | | ``scancel <jobid>``                                                                                                           | 
+--------------+---------------------------------------------------------------------------------------------------------------------------------+
| ``scontrol`` | | Used to view or modify job configuration.                                                                                     |
|              | | E.g., to place a job on hold:                                                                                                 |
|              | | ``scontrol hold <jobid>``                                                                                                     |  
+--------------+---------------------------------------------------------------------------------------------------------------------------------+



Vapor container guide
=====================


You can also build and run containers on Vapor with Apptainer. Vapor provides Apptainer v1.4.1. You
can build containers from Apptainer defintion files or pull images from a registry like Dockerhub. simplempich.def

Building and running OSU Benchmarks container example (with GCC and MPICH)
--------------------------------------------------------------------------

- Create a file named ``simplempich.def``

    .. code-block:: singularity 
    
        Bootstrap: docker
        From: opensuse/leap:15.6
        %environment
            # Point to MPICH binaries, libraries man pages
            export MPICH_DIR=/opt/mpich
            export PATH="$MPICH_DIR/bin:$PATH"
            export LD_LIBRARY_PATH="$MPICH_DIR/lib:$LD_LIBRARY_PATH"
            export MANPATH=$MPICH_DIR/share/man:$MANPATH
            # Point to rocm locations
            export ROCM_PATH=/opt/rocm
            export LD_LIBRARY_PATH="/opt/rocm/lib:/opt/rocm/lib64:$LD_LIBRARY_PATH"
            export PATH="/opt/rocm/bin:$PATH"
        
        %post
        echo "Installing required packages..."
        export DEBIAN_FRONTEND=noninteractive
        zypper install -y wget tar make sudo git fakeroot gzip gcc gcc-c++ gcc-fortran
        export MPICH_VERSION=3.4.2
        export MPICH_URL="http://www.mpich.org/static/downloads/$MPICH_VERSION/mpich-$MPICH_VERSION.tar.gz"
        export MPICH_DIR=/opt/mpich
        echo "Installing MPICH..."
        mkdir -p /mpich
        mkdir -p /opt
        # Download
        cd /mpich && wget -O mpich-$MPICH_VERSION.tar.gz $MPICH_URL && tar --no-same-owner -xzf mpich-$MPICH_VERSION.tar.gz
        # Compile and install
        cd /mpich/mpich-$MPICH_VERSION && ./configure --disable-fortran --with-device=ch4:ofi --prefix=$MPICH_DIR && make install
        rm -rf /mpich
        # Set env variables so we can compile our application
        
        export PATH=$MPICH_DIR/bin:$PATH
        export LD_LIBRARY_PATH=$MPICH_DIR/lib:$LD_LIBRARY_PATH
        echo "Compiling the MPI application..."
        cd /
        curl -o osubenchmarks-7.2.tar.gz https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-7.2.tar.gz && tar -xzf osubenchmarks-7.2.tar.gz --no-same-owner
        cd osu-micro-benchmarks-7.2 && ./configure CC=mpicc CXX=mpicc  && make  && rm ../osubenchmarks-7.2.tar.gz

- Build the container with

    .. code-block:: bash
    
        apptainer build simplempich.sif simplempich.def

Building and running OSU Benchmarks container example (with Intel and Intel MPI)
--------------------------------------------------------------------------------

If you want to build application in the container with the Intel Classic compiler and Intel MPI, you will need to install the appropriate version of the Intel OneAPI release, and set up several environment variables.

- First create the file ``intelenvs`` with the required environment variables. This file will be copied into the container image and will be sourced every time the container is started to set up the environment variables.

    .. code-block:: bash
    
        export INTEL_PATH=/opt/intel/oneapi/compiler/2023.2.0
        export INTEL_VERSION=2023.2.0
        export INTEL_COMPILER_TYPE=CLASSIC
        export LD_LIBRARY_PATH=/opt/intel/oneapi/mpi/2021.10.0/lib/release:/opt/intel/oneapi/compiler/2023.2.0/linux/lib:/opt/intel/oneapi/compiler/2023.2.0/linux/lib/x64:/opt/intel/oneapi/compiler/2023.2.0/linux/lib/oclfpga/host/linux64/lib:/opt/intel/oneapi/compiler/2023.2.0/linux/compiler/lib/intel64_lin:$LD_LIBRARY_PATH
        export CMAKE_PREFIX_PATH=/opt/intel/oneapi/compiler/2023.2.0/linux/IntelDPCPP:$CMAKE_PREFIX_PATH
        export NLSPATH=/opt/intel/oneapi/compiler/2023.2.0/linux/compiler/lib/intel64_lin/locale/%l_%t/%N:$NLSPATH
        export OCL_ICD_FILENAMES=libintelocl_emu.so:libalteracl.so:/opt/intel/oneapi/compiler/2023.2.0/linux/lib/x64/libintelocl.so
        export ACL_BOARD_VENDOR_PATH=/opt/intel/OpenCLFPGA/oneAPI/Boards
        export FPGA_VARS_DIR=/opt/intel/oneapi/compiler/2023.2.0/linux/lib/oclfpga
        export CMPLR_ROOT=/opt/intel/oneapi/compiler/2023.2.0
        export INTELFPGAOCLSDKROOT=/opt/intel/oneapi/compiler/2023.2.0/linux/lib/oclfpga
        export LIBRARY_PATH=/opt/intel/oneapi/mpi/2021.10.0/lib/release:/opt/intel/oneapi/mpi/2021.10.0/lib/:/opt/intel/oneapi/mpi/2021.10.0/lib/:/opt/intel/oneapi/compiler/2023.2.0/linux/compiler/lib/intel64_lin:/opt/intel/oneapi/compiler/2023.2.0/linux/lib:$LIBRARY_PATH
        export DIAGUTIL_PATH=/opt/intel/oneapi/compiler/2023.2.0/sys_check/sys_check.sh:$DIAGUTIL_PATH
        export MANPATH=/opt/intel/oneapi/compiler/2023.2.0/documentation/en/man/common:$MANPATH
        export PATH=/opt/intel/oneapi/compiler/2023.2.0/linux/bin/intel64:/opt/intel/oneapi/compiler/2023.2.0/linux/lib/oclfpga/bin:/opt/intel/oneapi/compiler/2023.2.0/linux/bin/intel64:/opt/intel/oneapi/compiler/2023.2.0/linux/bin:$PATH
        export PKG_CONFIG_PATH=/opt/intel/oneapi/compiler/2023.2.0/lib/pkgconfig:$PKG_CONFIG_PATH
        export LD_LIBRARY_PATH=/opt/intel/oneapi/mpi/2021.10.0/lib/:/opt/intel/oneapi/mkl/2023.2.0/lib/intel64:$LD_LIBRARY_PATH
        export CPATH=/opt/intel/oneapi/compiler/2023.2.0/linux/include:/opt/intel/oneapi/mkl/2023.2.0/include:$CPATH
        export NLSPATH=/opt/intel/oneapi/mkl/2023.2.0/lib/intel64/locale/%l_%t/%N:$NLSPATH
        export LIBRARY_PATH=/opt/intel/oneapi/mkl/2023.2.0/lib/intel64:$LIBRARY_PATH
        export MKLROOT=/opt/intel/oneapi/mkl/2023.2.0
        export PATH=/opt/intel/oneapi/mpi/2021.10.0/bin:/opt/intel/oneapi/mkl/2023.2.0/bin/intel64:$PATH
        export PKG_CONFIG_PATH=/opt/intel/oneapi/mkl/2023.2.0/lib/pkgconfig:$PKG_CONFIG_PATH
        export INCLUDE_PATH=/opt/intel/oneapi/mpi/2021.10.0/include:$INCLUDE_PATH
        export I_MPI_ROOT=/opt/intel/oneapi/mpi/2021.10.0

- Create the file ``simpleintelmpi.def``

    .. code-block:: singularity 
    
        Bootstrap: docker
        From: opensuse/leap:15.6
        
        %files
        ./intelenvs /intelenvs
        
        %environment
            # Point to MPICH binaries, libraries man pages
            export MPICH_DIR=/opt/mpich
            export PATH="$MPICH_DIR/bin:$PATH"
            export LD_LIBRARY_PATH="$MPICH_DIR/lib:$LD_LIBRARY_PATH"
            export MANPATH=$MPICH_DIR/share/man:$MANPATH
            # Point to rocm locations
            export ROCM_PATH=/opt/rocm
            export LD_LIBRARY_PATH="/opt/rocm/lib:/opt/rocm/lib64:$LD_LIBRARY_PATH"
            export PATH="/opt/rocm/bin:$PATH"
            source /intelenvs
        
        %post
        set -xe
        echo "Installing required packages..."
        export DEBIAN_FRONTEND=noninteractive
        zypper install -y wget tar make sudo git fakeroot gzip gcc gcc-c++ gcc-fortran which vim
        
        
        ## adding intel and internal cray pkg repos
        tee > /etc/zypp/repos.d/oneAPI.repo << EOF
        [oneAPI]
        name=Intel® oneAPI repository
        baseurl=https://yum.repos.intel.com/oneapi
        enabled=1
        gpgcheck=1
        repo_gpgcheck=1
        gpgkey=https://yum.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
        EOF
        
    
        zypper --releasever=15.6 --non-interactive --gpg-auto-import-keys  refresh
        ## installing intel 2023.2 since that is the version that has intel-classic 2021.10 (and 2023.2 is the last release that provides intel-classic)
        zypper --non-interactive --gpg-auto-import-keys install -y intel-dpcpp-cpp-compiler-2023.2.0  intel-oneapi-compiler-fortran-2023.2.0 intel-oneapi-mpi-devel-2021.10.0
        
        source /intelenvs
        which mpicc
        echo "Compiling the MPI application..."
        cd /
        curl -o osubenchmarks-7.2.tar.gz https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-7.2.tar.gz && tar -xzf osubenchmarks-7.2.tar.gz --no-same-owner
        cd osu-micro-benchmarks-7.2 && ./configure CC=mpiicc CXX=mpiicpc  && make  && rm ../osubenchmarks-7.2.tar.gz
    

- Build the container image with

    .. code-block:: bash
    
        apptainer build simpleintelmpi.sif simpleintelmpi.def


- To run the container, write a job script that will bind in the host's MPI libraries into the container. For example, create the below file ``submitbind.sl`` and submit the job with ``sbatch submitbind.sl``.

    .. code-block:: bash
    
        #!/bin/bash
        
        #SBATCH -A stf007uanofn
        #SBATCH -J test
        #SBATCH -N 2
        #SBATCH -o logs/subil_%j.out
        #SBATCH -t 01:00:00
        ###SBATCH --ntasks-per-node=16
        
        module reset
        module load oneapi
        
        
        export APPTAINERENV_LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/lib64/libibverbs::\$LD_LIBRARY_PATH"
        export APPTAINER_CONTAINLIBS="/usr/lib64/libjansson.so.4,/usr/lib64/libjson-c.so.5,/usr/lib64/libnl-3.so.200,/usr/lib64/libibverbs.so.1,/usr/lib64/libnuma.so.1,/usr/lib64/libnl-cli-3.so.200,/usr/lib64/libnl-genl-3.so.200,/usr/lib64/libnl-nf-3.so.200,/usr/lib64/libnl-route-3.so.200,/usr/lib64/libnl-3.so.200,/usr/lib64/libnl-idiag-3.so.200,/usr/lib64/libnl-xfrm-3.so.200,/usr/lib64/libnl-genl-3.so.200"
        export APPTAINER_BIND=/sw/vapor,/var/spool/slurmd,${PWD},/etc/libibverbs.d,/usr/lib64/libibverbs,/usr/lib64/libnl,${HOME}
        
        set -x
        
        srun --ntasks-per-node=16 apptainer exec --writable-tmpfs simplempich.sif /osu-micro-benchmarks-7.2/c//mpi/collective/blocking/osu_alltoall -m 4096
        srun --ntasks-per-node=16 apptainer exec --writable-tmpfs simpleintelmpi.sif /osu-micro-benchmarks-7.2/c//mpi/collective/blocking/osu_alltoall -m 4096
    
