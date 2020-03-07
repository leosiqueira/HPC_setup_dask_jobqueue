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

Although the examples on this page are target at using UMiami's `Triton`_ supercomputer (or `Pegasus`), the concepts here should generally apply to typical HPC systems. Furthermore, the steps above primarily work for performing other parallel computing at Triton (Pegasus) that do not use the Pangeo-like python ecosystem but use distributed computing with Dask. This document assumes that you already have access to an HPC system like Triton (Pegasus), and are comfortable using the command line. 

.. image:: /figures/bringiton.jpg
    :width: 100px
    :align: center
    :height: 50px
