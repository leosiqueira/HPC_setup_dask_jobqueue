.. _hpc:

Getting Started with Dask on HPC
==================================

This tutorial covers how to set up an environment to run operations in parallel using multicore processing on High-Performance Computing (HPC) systems. In particular, it covers the following:

1. Install `conda`_ and create an environment.
2. Configure `Jupyter`_.
3. Launch `Dask`_ with a job scheduler.
4. Launch a `Jupyter`_ server for your job.
5. Connect to `JupyterLab`_ and the `Dask-Dashboard`_ from your personal computer.

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
  
The Miniconda distribution packages together just ``python``, ``conda``, and a small number of other packages. Its download size is around 50MB or less than a tenth of the size of Anaconda distribution (100+ packages). Moreover, the conda tool is very valuable and what we will use to set up a robust environment. Download and install Miniconda for Triton (see notes for Pegasus version),

::

    cd ~/src
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-ppc64le.sh
    bash Miniconda3-latest-Linux-ppc64le.sh -bfp ~/local/miniconda3


.. note:: 

	*For Pegasus, use the following version:*
    
	::

		wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
		bash Miniconda3-latest-Linux-x86_64.sh -bfp ~/local/miniconda3
               
	*Moreover, for Pegasus, make sure you are not using the default python module by adding the following line to your*           ``~/.bashrc``,
    
    	::

        	module unload python

    	*and start another terminal, or type* ``source ~/.bashrc`` *to make this change effective for the current shell.* 
 
This installation comprises a self-contained Python environment, with *install prefix*
(location) ``~/local/miniconda3``, that we can manipulate safely without requiring the involvement of support services.
It also allows you to create isolated software environments so that we can experiment in the future safely. 

.. note::

    *You may choose to install miniconda in any other directory, e.g., 
    if your home space quota is small, by changing the install prefix.
    You may also run* ``bash Miniconda3-latest-*.sh`` *to go
    through the whole installation process if desired.*

Make the conda command available in all bash shells with,

::

	~/local/miniconda3/condabin/conda init
	
	
Note that this command modifies your user's ``~/.bashrc`` file. In addition,
the ``conda`` command being made available, the ``base`` conda environment is automatically
activated (i.e., environment variables set and all executables put on ``$PATH``). 

IMPORTANT: After running ``conda init``, your shells need to be closed and restarted, or type ``source ~/.bashrc``, to make  changes effective for the current shell.

Before creating your environment, we recommend updating your conda package manager with

::
    
    conda update conda

.. note:: 

    *Depending on if you chose to initialize Miniconda in your* ``~/.bashrc``
    *at the end of the installation process (or like in the above), a* ``conda update`` *activates a* ``base``
    *environment by default. If you wish to prevent conda from activating the* ``base``
    *environment at shell initialization (recommended), use*
    
    ::
    
            conda config --set auto_activate_base false
    
    *The above creates a* ``./condarc`` *in your home directory with this setting the first time you run it.*

Create a new conda environment for our pangeo work:

::

    conda create -n myenv -c conda-forge -y python=3.6 \
    nbserverproxy jupyterlab=2.0.0 nodejs dask_labextension \
    dask-jobqueue ipywidgets tornado==5.1.1

.. note::

   *Depending on your application, you may choose to remove or add conda
   packages to this list (Xarray includes Dask and Pandas packages as dependencies).*

To see a list of all of your environments, run:

::

  conda env list

To remove an environement,

::
  
  conda remove --name myenv --all

Activate your environment

::

    conda activate myenv

Your prompt should now look something like this (note the myenv environment name before the prompt):

::

    (myenv) $

And if you ask where your Python command lives, it should direct you to
somewhere in your home directory:

::

    (myenv) $ which python
    ~/local/miniconda3/envs/myenv/bin/python
    

::
	
	jupyter labextension install dask-labextension
	jupyter serverextension enable --py --sys-prefix dask_labextension

To move out of your environment,

::

    conda deactivate
    
Configure Jupyter
-----------------

The lastest `Jupyter`_ versions (v5.0 or newer) allows you to setup your password using

::
   
      jupyter notebook --generate-config
      jupyter notebook password

It  will prompt you for a password, and store the hashed password in your
``jupyter_notebook_config.json``.

You will need to set these two lines in ~/.jupyter/jupyter_notebook_config.py.

First to allow remote origins:

::

    c.NotebookApp.allow_origin = '*'

and second to listen on all IPs:

::

    c.NotebookApp.ip = '0.0.0.0'
   
For security reasons, we recommend making sure your ``jupyter_notebook_config.py``
is readable only by you. For more information on and other methods for
securing Jupyter, check out
`Securing a notebook server <http://jupyter-notebook.readthedocs.io/en/stable/public_server.html#securing-a-notebook-server>`__
in the Jupyter documentation.

::

    chmod 400 ~/.jupyter/jupyter_notebook_config.py

Finally, we want to configure dask's dashboard to forward through Jupyter,
instead of using ssh port forwarding. This can be done by editing the dask
distributed config file, e.g.: ``.config/dask/distributed.yaml``. By default
when ``dask.distributed`` and/or ``dask-jobqueue`` is first imported, it places
a file at ``~/.config/dask/distributed.yaml`` with a commented out version.
You can create this file and do this first import by simply 

::

    python -c 'from dask.distributed import Client'

In this ``.config/dask/distributed.yaml`` file, set:

.. code:: python

  #   ###################
  #   # Bokeh dashboard #
  #   ###################
  #   dashboard:
      link: "/proxy/{port}/status"

Further Reading
---------------

We have not attempted to provide a comprehensive tutorial on how to use Dask or Jupyter on HPC systems because each HPC system is uniquely configured. Instead, we have provided a friendly and generalizable workflow for deploying parallel multicore processing using python. Below we provide a few useful links for further customization of these tools.

 * `Deploying Dask on HPC <http://dask.pydata.org/en/latest/setup/hpc.html>`__
 * `Configuring and Deploying Jupyter Servers <http://jupyter-notebook.readthedocs.io/en/stable/index.html>`__

.. _conda: https://conda.io/docs/
.. _Jupyter: https://jupyter.org/
.. _JupyterLab: https://jupyterlab.readthedocs.io/en/stable/
.. _Dask: https://dask.pydata.org/
.. _Dask-Dashboard: https://docs.dask.org/en/latest/diagnostics-distributed.html
.. _dask-jobqueue: http://dask-jobqueue.readthedocs.io
