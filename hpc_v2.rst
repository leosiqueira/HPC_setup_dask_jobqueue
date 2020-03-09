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

    $ mkdir -p ~/src ~/local
  
The Miniconda distribution packages together just ``python``, ``conda``, and a small number of other packages. Its download size is around 50MB or less than a tenth of the size of Anaconda distribution (100+ packages). Moreover, the conda tool is very valuable and what we will use to set up a robust environment. Download and install Miniconda for Triton (see notes for Pegasus version),

::

    $ cd ~/src
    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-ppc64le.sh
    $ bash Miniconda3-latest-Linux-ppc64le.sh -bfp ~/local/miniconda3


.. note:: 

	*For Pegasus, use the following version:*
    
	::

		$ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
		$ bash Miniconda3-latest-Linux-x86_64.sh -bfp ~/local/miniconda3
               
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

	$ ~/local/miniconda3/condabin/conda init
	
	
Note that this command modifies your user's ``~/.bashrc`` file. In addition,
the ``conda`` command being made available, the ``base`` conda environment is automatically
activated (i.e., environment variables set and all executables put on ``$PATH``). 

IMPORTANT: After running ``conda init``, your shells need to be closed and restarted, or type ``source ~/.bashrc``, to make  changes effective for the current shell.

Before creating your environment, we recommend updating your conda package manager with

::
    
    $ conda update conda

.. note:: 

    *Depending on if you chose to initialize Miniconda in your* ``~/.bashrc``
    *at the end of the installation process (or like in the above), a* ``conda update`` *activates a* ``base``
    *environment by default. If you wish to prevent conda from activating the* ``base``
    *environment at shell initialization (recommended), use*
    
    ::
    
           $ conda config --set auto_activate_base false
    
    *The above creates a* ``./condarc`` *in your home directory with this setting the first time you run it.*

Create a new conda environment for your work:

::

    $ conda create -n myenv -c conda-forge -y python=3.6 \
      nbserverproxy jupyterlab=2.0.0 nodejs dask_labextension \
      dask-jobqueue ipywidgets tornado==5.1.1

.. note::

	*IMPORTANT: For setting the diagnostics dashboard via web interface at UMiami-Triton (Pegasus) in an 			interactive (or regular) scheduled job, we need to downgrade the Tornado package for now due to an issue in V6.0. 	  Moreover, depending on your application, you may choose to remove or add conda packages to this list. For earth 	  sciences studies, Xarray is a useful choice, includind Dask and Pandas packages as dependencies, and is usually 	  combined with Scipy, Cartopy, among others.*

To see a list of all of your environments, run:

::

  $ conda env list

To remove an environment,

::
  
  $ conda remove --name myenv --all

Let's activate your environment with,

::

   $ conda activate myenv

Your prompt should now look something like this (note the myenv environment name before the prompt):

::

    (myenv) $

And if you ask where your Python command lives, it should direct you to
somewhere in your home directory:

::

    (myenv) $ which python
    ~/local/miniconda3/envs/myenv/bin/python
    

To move out of your environment,

::

    (myenv) $ conda deactivate
    
.. note::

	*see* `Managing Environments <https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>`__ *for more information.*
	
Configure Jupyter
-----------------

The lastest `Jupyter`_ versions (v5.0 or newer) allows you to set up your password using

::
   
      (myenv) $ jupyter notebook --generate-config
      (myenv) $ jupyter notebook password

It  prompts you to create a password for the Jupyter server, and store the hashed password in your
``jupyter_notebook_config.json``.

You also need to uncomment and set these two lines in ``~/.jupyter/jupyter_notebook_config.py``.

First to allow remote origins:

::

    c.NotebookApp.allow_origin = '*'

and second to listen on all IPs:

::

    c.NotebookApp.ip = '0.0.0.0'
   
For security reasons, we recommend making sure your ``jupyter_notebook_config.py``
is readable only by you with,

::

    (myenv) $ chmod 400 ~/.jupyter/jupyter_notebook_config.py

.. note::
*For more information on and other methods for securing Jupyter, check out*
`Securing a notebook server <http://jupyter-notebook.readthedocs.io/en/stable/public_server.html#securing-a-notebook-server>`__ *in the Jupyter documentation.*

Finally, we want to configure dask's dashboard to forward through JupyterLab,
instead of using ssh port forwarding. This can be done by editing the dask
distributed config file, e.g., ``.config/dask/distributed.yaml``. By default
when ``dask.distributed`` and/or ``dask-jobqueue`` is first imported, it places
a file at ``~/.config/dask/distributed.yaml`` with a commented out version.
You can create this file and do this first import by simply running,

::

    (myenv) $ python -c 'from dask.distributed import Client'

In this ``.config/dask/distributed.yaml`` file, set:

.. code:: python

  #   ###################
  #   # Bokeh dashboard #
  #   ###################
  #   dashboard:
      link: "/proxy/{port}/status"

------------

Basic and friendly deployment: Jupyter + dask-jobqueue
----------------------------------------

Start a Jupyter Notebook Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we have Jupyter configured, we can start a JupyterLab (or notebook) server. In many
cases, your system administrators require want you to run this notebook server in
an interactive session on a compute node. Please kindly refrain from running
resource-intensive jobs on the UMiami-Triton(Pegasus) login nodes, unless you have direct access to dedicated
compute nodes like Umiami-Visx. Submit your production
jobs to LSF, and use the interactive queue – **not the login nodes** – for
resource-intensive command-line processes. You may compile and test jobs on
login nodes in any case. However, any jobs exceeding 30 minutes of run time or using excessive
resources on the login nodes will be terminated, and the UM-IDSC account responsible
for those jobs may be suspended. This is not a universal rule, but it is
one we'll follow for this tutorial.

If you are using dask-jobqueue within Jupyter, one user-friendly solution to see the
Diagnostics Dashboard is to use ``nbserverproxy`` or ``dask-labextension``. As the dashboard HTTP endpoint is 
launched inside the same node as Jupyter, this is the solution for viewing it at
UMiami-Triton (Pegasus) when running within an interactive job. You just need to have it installed
in the Python environment you use for launching JupyterLab (or notebook), and activate it. First, we need to install the JupyterLab extension to manage Dask clusters, as well as embed Dask's dashboard plots directly into JupyterLab panes with,

::
	
	(myenv) $ jupyter labextension install dask-labextension
	Building jupyterlab assets (build:prod:minimize)


Then enable the extension for JupyterLab with,

::

	(myenv) $ jupyter serverextension enable --py --sys-prefix dask_labextension
	Enabling: dask_labextension
	- Writing config: /home/$USER/local/miniconda3/envs/myenv/etc/jupyter
    	- Validating...
      	dask_labextension 0.3.3 OK

In our case, the Triton (Pegasus) supercomputer uses the LSF job scheduler, so within your (myenv)
environment typing

::

	(myenv) $ bsub -J jupyter -Is -q interactive jupyter lab --no-browser --ip=0.0.0.0 --port=8888
	Job <33565> is submitted to queue <interactive>.
	<<Waiting for dispatch ...>>
	<<Starting on t037>>
	[LabApp] JupyterLab extension loaded from 	
	~/local/miniconda3/envs/myenv/lib/python3.6/site-packages/jupyterlab
	[LabApp] JupyterLab application directory is ~/local/miniconda3/envs/myenv/share/jupyter/lab
	[LabApp] Serving notebooks from local directory: /home/$USER
	[LabApp] The Jupyter Notebook is running at:
	[LabApp] http://t037:8888/
	[LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
	
which gives us an interactive job on the ``interactive`` queue for 6 hours running JupyterLab server in node ``t037``.

Now, connect to the server using a ssh tunnel from your local machine
(this could be your laptop or desktop).

::

    $ ssh -N -L localhost:8890:t037:8888  username@hpc_domain

You may need to change the details in the command above, but the basic idea is
that we're passing the port 8888 from the compute node ``t037`` to your
local system port 8890. Now open http://localhost:8890 on your local machine browser, you should
find a JupyterLab server running!


.. note::
  
  *Sometimes, the Jupyter server and ssh port forwarding from the computing node may freeze and the user has to first kill    	the interacitve job, open another terminal and check its id number with* ``bjobs`` *and use* ``bkill`` *. Then find the     local machine PID linked with that port using*
  
  ::
  
    lsof -i:8890
  
  *Kill the ssh process with* ``kill PID``. *Redo the job submission step and port forwarding. Usually this happens at the    	very beggining of the session, once it is further established it rarely freezes.* 

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
