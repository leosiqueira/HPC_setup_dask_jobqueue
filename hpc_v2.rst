.. _hpc:

Getting Started with Dask and Pangeo on HPC
==================================

This tutorial covers how to set up an environment to run operations in parallel using multicore processing on High
Performance Computing (HPC) systems. In particular, it covers the following:

1. Install `conda`_ and creating an environment
2. Configure `Jupyter`_
3. Launch a `JupyterLab`_ server for your job
4. Launch `Dask`_ with a job scheduler
5. Connect to `Jupyter`_ and the `Dask`_ Dashboard from your personal computer

Although the examples on this page are target at using UMiami's `Triton <https://idsc.miami.edu/triton/>`__ supercomputer (or `Pegasus <https://idsc.miami.edu/pegasus/>`__), the concepts here should generally apply to typical HPC systems. Furthermore, the steps above primarily work for performing other parallel computing at Triton (Pegasus) that do not use the Pangeo-like python ecosystem but use distributed computing with Dask. This document assumes that you already have access to an HPC system like Triton (Pegasus), and are comfortable using the command line. 

.. image:: /figures/bringiton.jpg
    :width: 100px
    :align: center
    :height: 50px

Let's log into your HPC system and get started.

Installing a software environment
---------------------------------

Start with creating some directories,

::

    mkdir -p ~/src ~/local
  
The Miniconda distribution packages together just ``python``, ``conda``, and a small number of other packages. Its download size is around 50MB or less than a tenth the size of Anaconda distribution (100+ packages). Morevover, the conda tool is very valuable and what we will use to set up a robust environment. Download and install Miniconda for Triton (see notes for Pegasus version),

::

    cd ~/src
	wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-ppc64le.sh
	bash Miniconda3-latest-Linux-ppc64le.sh -bfp ~/local/miniconda3


.. note:: 

	*For Pegasus, use the following version:*
    
	::
        
	    wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
        bash Miniconda3-latest-Linux-x86_64.sh -b -p ~/local/bin/miniconda3
            
        
    *Moreover, for Pegasus, make sure you are not using the default python module by adding the following line to your*           ``~/.bashrc``,
    
    ::

        module unload python

    *and start another terminal, or type* ``source ~/.bashrc`` *to make this change effective.* 
 
This installation comprises a self-contained Python environment, with *install prefix*
(location) ``~/local/miniconda3``, that we can manipulate safely without requiring the involvement of support services.
It also allows you to create isolated software environments so that we can experiment in the future safely. 

.. note:: 

    *You may choose to install miniconda in any other directory, e.g., 
    if your home space quota is small, by changing the install prefix.
    You may also run* ``bash Miniconda3-latest-*.sh`` *to go
    trough the whole installation process if desired.*
 
