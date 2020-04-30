.. _hpc:

Getting Started with Dask on HPC
==================================

This tutorial covers how to set up an environment to run operations in parallel using multicore processing on High-Performance Computing (HPC) systems. In particular, it covers the following:

1. Install `conda`_ and create an environment.
2. Configure `Jupyter`_.
3. Launch `Dask`_ with a job scheduler.
4. Launch a `Jupyter`_ server for your job.
5. Connect to `JupyterLab`_ and the `Dask-Dashboard`_ from your personal computer.

Although the examples on this page are target at using UMiami's `Triton <https://idsc.miami.edu/triton/>`__ supercomputer, the concepts here should generally apply to typical HPC systems. Furthermore, the steps above primarily work for performing other parallel computing in Triton that do not use an extensive python ecosystem but use distributed computing with Dask. This document assumes that you already have access to an HPC system like Triton, and are comfortable using the command line.

.. image:: /figures/bringiton.jpg
    :width: 100px
    :align: center
    :height: 50px

Disclaimer: The views, information, or opinions expressed in this tutorial are solely those of the author and do not necessarily represent those of University of Miami, University of Miami - Institute for Data Science and Computing (IDSC), and its employees (see oficial documentation at https://acs-docs.readthedocs.io/index.html).

Let's log into your HPC system and get started.

Installing a software environment
---------------------------------

Start with creating some directories,

::

    $ mkdir -p ~/src ~/local
  
The Miniconda distribution packages together just ``python``, ``conda``, and a small number of other packages. Its download size is around 50MB or less than a tenth of the size of Anaconda distribution, which has 100+ packages. Moreover, the conda tool is very valuable and what we will use to set up a robust environment. Download and install Miniconda for Triton,

::

    $ cd ~/src
    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-ppc64le.sh

.. note::

    *Triton is an IBM Power System (pp64le) and not a x86 architecture.*

It is recommended to verify the download integrity with md5sum by running,
 
::

    $ md5sum Miniconda3-latest-Linux-ppc64le.sh
    
and checking the website (https://repo.anaconda.com/miniconda/) for its valid hash value so that you can make sure the download completed correctly. Then, you can run the installer by including the ``bash`` command regardless of whether or not you are using Bash shell,

::

    $ bash Miniconda3-latest-Linux-ppc64le.sh -bfp ~/local/miniconda3
 
This installation comprises a self-contained Python environment, with *install prefix*
(location) ``~/local/miniconda3``. Here, we want to manage your own Miniconda, threfore it's installed in your home directory so that we can manipulate safely without requiring the involvement of support services.
It also allows you to create isolated software environments so that we can experiment in the future safely. 

.. note::

    *You may choose to install miniconda in any other directory, e.g., 
    if your home space quota is small, by changing the install prefix.
    You may also run* ``bash Miniconda3-latest-*.sh`` *to go
    through the whole installation process (and prompts) if desired.*

In our quick (and promptless) installation we need to initialize the manager after the installation process is done, first run ``source <path to conda>/bin/activate`` and then run ``conda init`` to make the ``conda`` command available in all Bash shells with,

::

	$ source ~/local/miniconda3/bin/activate
	(base) $ conda init
	
	
Note that this command modifies your user's ``~/.bashrc`` file. In addition,
the ``conda`` command being made available, the ``base`` conda environment is automatically
activated (i.e., environment variables set and all executables put on ``$PATH``). 

Before creating your environment, we recommend updating your conda package manager with

::
    
    $ conda update conda

.. note:: 

    *Depending on if you chose to initialize Miniconda in your* ``~/.bashrc``
    *at the end of the installation process (or like in the above), a* ``conda init`` *activates a* ``base``
    *environment by default. If you wish to prevent conda from activating the* ``base``
    *environment at shell initialization, so that each shell session will not have the base environment activated (recommended), use*
    
    ::
    
           $ conda config --set auto_activate_base false
	   
    
    *The above creates a* ``./condarc`` *in your home directory with this setting the first time you run it.*

At this point, before creating your first environment we can add the *conda-forge* channel that contains repositories of *conda* recipes with,

::
    
	$ conda config --add channels conda-forge
	
	  
This keeps your *default* channel, but puts *conda-forge* high in priority, so that install packages will be searched on *conda-forge* before going to *default*. 

.. note::

	*The conda-forge and defaults channels are not 100% compatible and this can lead to errors when the install environment mixes packages from multiple channels. The approach here is to ensure that the dependencies will come from the conda-forge channel (see also notes on using* ``pip`` *to install packages).*

Create a new conda environment for your work (here named *myenv*):

::

    $ conda create -n myenv -c conda-forge -y python=3.7 \
      jupyterlab=2.0.0 nodejs nbserverproxy ipywidgets dask-jobqueue numpy
      

.. note::

	*The above creates a conda environment with the latest Python 3.7.x and five packages needed for the examples below. It will resolve the dependencies altogether and avoid further conflicts (recommended). The environment will be created at ``~/.conda/envs``. Depending on your application, you may choose to remove or add conda packages to this list. For earth sciences studies, for example, Xarray is a useful choice, wich includes Dask and Pandas packages as dependencies, and is usually combined with Scipy, Cartopy, among others:*
	
	::
	
	    $ conda create -n <environment name> python=<version> <package1> <package2> <...>
	

To see a list of all of your environments, run:

::

  $ conda env list

To remove an environment,

::
  
  $ conda remove --name myenv --all

Let's activate your environment with,

::

   $ conda activate myenv

Your prompt should now look something like this (note the ``(<environment>)`` name before the prompt):

::

    (myenv) $

And if you ask where your Python command lives, it should direct you to
somewhere in your *home* (or *install prefix*) directory:

::

    (myenv) $ which python
    ~/local/miniconda3/envs/myenv/bin/python
    

To move out of your environment,

::

    (myenv) $ conda deactivate
    
.. note::

	*see* `Managing Environments <https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>`__ *for more information.*
	
	
To install packages in your environment,

::

   (myenv)$ conda install <package>


or ``conda install <package>=<version>`` if you want a specific version. If conda finds the package from the channels listed in the ``./condarc`` file, it will download and install the package, otherwise you can search in Anaconda Cloud and choose Platform ``linux-ppc64le`` (for UMiami-Triton). Click on the name of the selected package, and the detail page will show you the specific channel to install from, for example,

::

	(<environment>)$ conda install -c <channel> <package>
	

If the package is still not found, you may try

::

	(<environment>)$ pip install <package>

.. note::

	*Issues may arise when using pip and conda together to install packages. You rather use an isolated conda environment, and only after conda has been used to install as many packages as possible should pip be used to install any remaining software. If further modifications are needed to the environment, it is best to create a new environment rather than running conda after pip.*	
	
Configure Jupyter
-----------------

The lastest `Jupyter`_ versions (v5.0 or newer) allows you to set up your password using

::
   
      (myenv) $ jupyter notebook --generate-config
      		Writing default config to: /home/$USER/.jupyter/jupyter_notebook_config.py
      (myenv) $ jupyter notebook password
      		Wrote hashed password to /home/$USER/.jupyter/jupyter_notebook_config.json

It  prompts you to create a password for the Jupyter server, and store the hashed password in your
``jupyter_notebook_config.json``.

You will also need to uncomment and set these three lines in ``~/.jupyter/jupyter_notebook_config.py``.

First to allow remote origins,

::

    c.NotebookApp.allow_origin = '*'

then,    

::

    c.NotebookApp.allow_remote_access = True

and last, listen on all IPs:

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

In this ``~/.config/dask/distributed.yaml`` file, set:

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
cases, your system administrators require you to run this notebook server in
an interactive session on a compute node. Please kindly refrain from running
resource-intensive jobs on the UMiami-Triton login nodes, unless you have direct access to dedicated
compute nodes. Submit your production
jobs to LSF, and use the interactive queue – **not the login nodes** – for
resource-intensive command-line processes. You may compile and test jobs on
login nodes in any case. However, any jobs exceeding 30 minutes of run time or using excessive
resources on the login nodes will be terminated, and the UM-IDSC account responsible
for those jobs may be suspended. This is not a universal rule, but it is
one we'll follow for this tutorial.

If you are using dask-jobqueue within Jupyter, one user-friendly solution to see the
Diagnostics Dashboard is to use ``nbserverproxy`` or ``dask-labextension``. As the dashboard HTTP endpoint is 
launched inside the same node as Jupyter, this is the solution for viewing it at
UMiami-Triton when running within an interactive job. You just need to have it installed
in the Python environment you use for launching JupyterLab (or notebook), and activate it,

::

	(myenv) $ jupyter serverextension enable --py nbserverproxy
	Enabling: nbserverproxy
	- Writing config: /home/$USER/.jupyter
   		- Validating...
     		nbserverproxy  OK

Then, we need to install the JupyterLab extension to manage Dask clusters, as well as embed Dask's dashboard plots directly into JupyterLab panes uisng ``pip`` with,

::

	pip install dask_labextension

and install the extension for Jupyter,

::
	
	(myenv) $ jupyter labextension install dask-labextension
	Building jupyterlab assets (build:prod:minimize)


Then enable the extension for JupyterLab with,

::

	(myenv) $ jupyter serverextension enable --py --sys-prefix dask_labextension
	Enabling: dask_labextension
	- Writing config: /home/$USER/local/miniconda3/envs/myenv/etc/jupyter
    	- Validating...
      	dask_labextension 2.0.1 OK

.. note::

*Standard online guidelines for enabling dask-labextension did not work in UMiami-Triton, changed to what works for now.*

Another extension install, 

::

	(myenv) $ jupyter labextension install @jupyter-widgets/jupyterlab-manager
	(myenv) $ jupyter lab clean
	
This command defaults to installing the latest version of the ``ipywidgets`` JupyterLab extension and ensure a clean reinstall of the JupyterLab extension.

In our case, the Triton supercomputer has an ``interactive`` queue, so within your (myenv)
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
  
  *Sometimes, the Jupyter server and ssh port forwarding from the computing node may freeze and the user has first to kill    	the interacitve job, open another terminal and check its id number with* ``bjobs`` *and use* ``bkill`` *. Then find the     local machine PID linked with that port using*
  
  ::
  
    lsof -i:8890
  
  *Kill the ssh process with* ``kill PID``. *Redo the job submission step and port forwarding. Usually, this happens at the very beginning of the session, once it is further established it rarely freezes.*
  
Launch Dask and dask-jobqueue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
From within your JupyterLab you can start a local cluster by creating a Python3 Jupyter Notebook and run in the first cell

.. code:: python

    from dask.distributed import Client, LocalCluster
    cluster = LocalCluster(n_workers=16, memory_limit='2.5GB', processes=True, threads_per_worker=4)
    client = Client(cluster)
    client

and it will output,

.. image:: /figures/cluster.jpg
    :width: 100px
    :align: center
    :height: 50px


Triton (IBM POWER System AC922) has at least 16 cores per processor, so the rule of thumb for threads per Dask worker is to choose the square root of the number of cores per processor. For Triton for example, this would mean that one could assign 4 to 5 threads per worker. We discuss the choice of workers, threads, and dask chunksize in a separate example (see `Monte Carlo estimate of Pi example <https://github.com/leosiqueira/HPC_ssh_keys_setup_dask_jobqueue/MC_example.html>`__).

To access the Diagnostics Dashboard you may open a separate tab an go to ``http://localhost:8890/proxy/8787/status`` or have the diagnostics embeded in the JupyterLab panes. For the latter, you click on the ``dasklab-extension`` symbol on the left-hand sidebar and paste ``http://localhost:8890/proxy/8787`` in the ``DASK BASHBOARD URL`` field. The grey buttons become available (orange) and they open new panes within JupyterLab like below, 

.. image:: /figures/embed_dashboard.jpg
    :width: 100px
    :align: center
    :height: 50px

Let's test Dask using the basic example below by running the following cells. We'll create a 2D array with 75000*75000 elements and 45GB in size (slightly larger than the total 40GB memory available),

.. code:: python

	import dask.array as da
        import numpy as np
	
.. code:: python

	size_in_bytes = 0.6e6
	size = int(size_in_bytes / 8)
	data = da.random.uniform(0, 1, size=( size , size ), chunks=(5000, 5000))
	data

.. image:: /figures/example.jpg
    :width: 100px
    :align: center
    :height: 50px
    
then compute the approximate singular value decomposition of this large array on the next cell. This algorithm is generally faster than the normal algorithm, but does not provide exact results, where ``k`` is the rank of the desired compressed rank-k SVD decomposition, 

.. code:: python

	u, s, v = da.linalg.svd_compressed(data, k= 10) # Randomly compressed rank-k SVD.

Up to this point, no computation was done (`lazy execution of code <https://tutorial.dask.org/01x_lazy.html>`__). You may call ``v.compute()`` when you want your right singular vector result as a NumPy array. If you started ``Client()`` above then you may want to watch the status page (or the task progress, stream, or worker memory panel) of the diagnostics dashboard during computation. You may check your SVD decompostion,

.. code:: python

	print("Left Singular Vectors:")
	print(u,  "\n")

	print("Singular Values:") 
	print(np.diag(s), "\n")

	print("Right Singular Vectors:") 
	print(v,  "\n")
	
	Left Singular Vectors:
	dask.array<getitem, shape=(75000, 10), dtype=float64, chunksize=(5000, 10), chunktype=numpy.ndarray> 

	Singular Values:
	dask.array<diag, shape=(10, 10), dtype=float64, chunksize=(10, 10), chunktype=numpy.ndarray> 

	Right Singular Vectors:
	dask.array<getitem, shape=(10, 75000), dtype=float64, chunksize=(10, 5000), chunktype=numpy.ndarray> 

If you have the available RAM for your dataset (here is quite tight for the computation) then you can persist data in memory. This allows future computations to be much faster, for example,

.. code:: python

	v = v.persist()

If you don't, you may save intermediate results to disk and then load them again for further computations. Again, in theory, Dask should be able to do the computation in a streaming fashion, but in practice this is a fail case for the Dask scheduler, because it tries to keep every chunk of an array that it computes in memory.

Triton Dask_jobqueue, LSFCluster, and JupyterHub
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TODO

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
