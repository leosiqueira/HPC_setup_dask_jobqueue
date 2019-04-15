.. _hpc:

Getting Started with Dask and Pangeo on HPC
==================================

This tutorial covers how to set up an environment to run operations in parallel using multicore processing on High
Performance Computing (HPC) systems. In particular it covers the following:

1. Install `conda`_ and creating an environment
2. Configure `Jupyter`_
3. Launch `Dask`_ with a job scheduler
4. Launch a `Jupyter`_ server for your job
5. Connect to `Jupyter`_ and the `Dask`_ dashboard from your personal computer

Although the examples on this page were developed using UM's `Pegasus`_ super
computer, the concepts here should be generally applicable to typical HPC systems. Furthermore, the steps above essentially work for performing other parallel computing at Pegasus that do not use Pangeo but use distributed computing with Dask.
This document assumes that you already have an access to an HPC like Pegasus,
and are comfortable using the command line. It may be necessary to work with your
system administrators to properly configure these tools for your machine.

You should log into your HPC system now.

Installing a software environment
---------------------------------

After you have logged into your HPC system, create some directories,

::

    mkdir -p ~/src ~/local/bin
  
download and install Miniconda,

::

    cd ~/src
    wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
    chmod +x Miniconda3-latest-Linux-x86_64.sh
    ./Miniconda3-latest-Linux-x86_64.sh

This comprises a self-contained Python environment that we can manipulate
safely without requiring the involvement of IT. It also allows you to
create isolated software environments so that we can experiment in the
future safely. Before creating your environment, we recommend you update
your conda package manager with

::
    
    conda update conda
    
.. note:: 

    *Depending if you chose to initialize Miniconda in your* ``~/.bashrc``
    *at the end of the installation, this new conda update will activate
    a* ``(base)`` *environment by default. If you wish to prevent conda
    from activating the* ``(base)`` *environment at shell initialization:*
    
    ::
    
            conda config --set auto_activate_base false
    
    *This will create a* ``./condarc`` *in your home
    directory with this setting the first time you run it.*

Create a new conda environment for our pangeo work:

::

    conda create -n pangeo -c conda-forge \
        python=3.6 jupyterlab nbserverproxy \
        mpi4py dask-jobqueue ipywidgets \
        xarray scipy netcdf4 matplotlib cartopy

.. note::

   *Depending on your application, you may choose to remove or add conda
   packages to this list.*

To see a list of all of your environments, run:

::

  conda info --envs

or

::

  conda env list

Activate the pangeo environment

::

    conda activate pangeo

Your prompt should now look something like this (note the pangeo environment name):

::

    (pangeo) $

And if you ask where your Python command lives, it should direct you to
somewhere in your home directory:

::

    (pangeo) $ which python
    /home/username/miniconda3/envs/pangeo/bin/python
    
Configure Jupyter
-----------------

When using recent Jupyter iteration (v5.0 or newer) you can setup your password using

::
   
      jupyter notebook --generate-config
      jupyter notebook password

It  will prompt you for a password, and store the hashed password in your
``jupyter_notebook_config.json``.
   
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

.. note::
  
  *This is an important step for setting the diagnostics dashboard via
  web interface at UM-Pegasus when running an interactive job.*  
------------

Basic and friendly deployment: Jupyter + dask-jobqueue
----------------------------------------

Start a Jupyter Notebook Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we have Jupyter configured, we can start a notebook server. In many
cases, your system administrators will want you to run this notebook server in
an interactive session on a compute node. Please kindly refrain from running
resource-intensive jobs on the UM-Pegasus login nodes. Submit your production
jobs to LSF, and use the interactive queue – **not the login nodes** – for
resource-intensive command-line processes. You may compile and test jobs on
login nodes. However, any jobs exceeding 30 minutes of run time or using excessive
resources on the login nodes will be terminated and the UM-CCS account responsible
for those jobs may be suspended. This is not universal rule, but it is
one we'll follow for this tutorial.

If you are using dask-jobqueue within Jupyter, one user friendly solution to see the
Diagnostics Dashboard is to use nbserverproxy. As the dashboard HTTP end point is 
launched inside the same node as Jupyter, this is the solution for viewing it at
UM-Pegasus when running within an interactive job. You just need to have it installed
in the Python environment you use for launching the notebook, and activate it,

::

    jupyter serverextension enable --py nbserverproxy
    ...
    Enabling: nbserverproxy
    - Writing config: /nethome/$user/.jupyter
    - Validating...
      nbserverproxy  OK

Then, once started, the dashboard will be accessible from your notebook URL by adding
the path ``/proxy/8787/status``, replacing 8787 by any other port you use or the dashboard
is bind to if needed. Sor for example:
::

http://localhost:8888/proxy/8787/status

with the example below.

In our case, the Pegasus super computer uses the LSF job scheduler, so typing:

::

  bsub -J jupyter -Is -q interactive jupyter notebook --no-browser --ip=0.0.0.0 --port=8888
  ...
  Job is submitted to <project> project.
  Job <20199271> is submitted to queue <interactive>.
  <<Waiting for dispatch ...>>
  <<Starting on n003>>
  [I 18:14:28.339 NotebookApp] JupyterLab extension loaded from /nethome/$USER/local/bin/miniconda3/envs/pangeo/lib/python3.6/site-packages/jupyterlab
  [I 18:14:28.339 NotebookApp] JupyterLab application directory is /nethome/$USER/local/bin/miniconda3/envs/pangeo/share/jupyter/lab
  [I 18:14:28.342 NotebookApp] Serving notebooks from local directory: /nethome/$USER
  [I 18:14:28.342 NotebookApp] The Jupyter Notebook is running at:
  [I 18:14:28.342 NotebookApp] http://(n003 or 127.0.0.1):8888/
  [I 18:14:28.342 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
  
This will get us an interactive job on the `interactive` queue for 6 hours running jupyter server.  

Now, connect to the server using an ssh tunnel from your local machine
(this could be your laptop or desktop).

::

    $ ssh -N -L localhost:8888:n003:8888  username@hpc_domain

You'll want to change the details in the command above but the basic idea is
that we're passing the port 8888 from the compute node `n003` to our
local system. Now open http://localhost:8888 on your local machine, you should
find a jupyter server running!

To access the Diagnostics Dashboard you open http://localhost:8888/proxy/8787/status.
The Dashboard has limited functionality at Pegasus nodes for now (help is welcome),
but logs and workers load are available.

.. note::
  
  *Sometimes at Pegasus, the jupyter server and ssh port forwarding from the computing node
  may freeze and the user has to first kill the interacitve job, check its id number with* 
  ``bjobs`` *, and find the local machine PID linked with that port using*
  
  ::
  
    lsof -i:8888
  
  *then kill the ssh process with* ``kill PID``. *Redo the job submission step and 
  port forwarding. Usually this happens at the very beggining of the session, once it is
  further established it doens't freeze.*  
  
  Further Reading
---------------

We have not attempted to provide a comprehensive tutorial on how to use Pangeo,
Dask, or Jupyter on HPC systems. This is because each HPC system is uniquely
configured. Instead we have provided a friendly and generalizable workflows 
for deploying Pangeo. Below we provide a few useful links for further
customization of these tools.

 * `Deploying Dask on HPC <http://dask.pydata.org/en/latest/setup/hpc.html>`__
 * `Configuring and Deploying Jupyter Servers <http://jupyter-notebook.readthedocs.io/en/stable/index.html>`__

.. _conda: https://conda.io/docs/
.. _Jupyter: https://jupyter.org/
.. _Dask: https://dask.pydata.org/
.. _Pegasus: http://ccs.miami.edu/pegasus
.. _dask-jobqueue: http://dask-jobqueue.readthedocs.io
