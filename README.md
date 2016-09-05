#Modified batchspawner for Jupyterhub [![Build Status](https://travis-ci.org/jupyterhub/batchspawner.svg?branch=master)](https://travis-ci.org/jupyterhub/batchspawner)
This is a custom spawner for Jupyterhub that is designed for installations on clusters using batch scheduling software. This version of batchspawner is specificly modified to work with [University of Memphis](http://www.memphis.edu/hpc/configuration.php) HPC environment.

This is the fork of original [batchspawner](https://github.com/jupyterhub/batchspawner). The target HPC has couple features which made batchspawner not function properly. 
* The login node does not resolve the computing nodes by dns name and computing nodes do not resolve login node as well
* The login node and the computing node does not return the job status in XML format, but batchspawner looks for XML  
* UofM HPC uses torque for resource management.

To address the above features, the modification is made in two places. 
* Instead of dns names, the nodes are represented by their IP address.
* In the command of job status, we added force option to return XML status. 

This package also includes WrapSpawner and ProfilesSpawner, which provide mechanisms for runtime configuration of spawners. There is no modification yet in these packages. 

We assume that you have installed anaconda or any virtual local environment with python version > 3.4.2

##Installation
1. get the package from github 
   ```bash 
   $ git clone git@github.com:farukahmedatgithub/batchspawner.git
   ```

2. from root directory of this repo (where setup.py is), run i
   ```bash 
   $ pip install -e  .
   ```
   If you don't actually need an editable version, you can simply run 
      
   ```bash
   $ pip install  https://github.com/farukahmedatgithub/batchspawner.git      (it did not work for me)
   ```
3. install jupyterhub follow [this](https://github.com/jupyterhub/jupyterhub) 
4. generate jupyterhub configuration file (default configuration file name is jupyterhub_config.py) 
   ```bash 
   jupyterhub --generate-config
   ```
5. add or enable lines in jupyterhub_config.py for the spawner you intend to use, e.g.
   
   ```python
      c.JupyterHub.spawner_class = 'batchspawner.TorqueSpawner'
   ```
3. Depending on the spawner, additional configuration will likely be needed.

I am posting the configuration here 


### Example minimum configuration

```python

# Set the log level by value or name.
c.Application.log_level = 'DEBUG' # 0,10,20.... 

# Allows ahead-of-time generation of API tokens for use by services.
c.JupyterHub.api_tokens = {'<put your token>':'<put your hpc user name>'}

# The ip for this process
c.JupyterHub.hub_ip = 'xx.xx.xx.xx' # put your login node ip which is facing towards the computing node

# Loaded from the CONFIGPROXY_AUTH_TOKEN env variable by default.
c.JupyterHub.proxy_auth_token = '<you put your auth token>'

# The class to use for spawning single-user servers.
# 
# Should be a subclass of Spawner.
c.JupyterHub.spawner_class = 'batchspawner.TorqueSpawner'

# Path to SSL certificate file for the public facing interface of the proxy
# 
# Use with ssl_key
c.JupyterHub.ssl_cert = '<path to your crt file>'

# Path to SSL key file for the public facing interface of the proxy
# 
# Use with ssl_cert
c.JupyterHub.ssl_key = '<path to your key file>'

# Once a server has successfully been spawned, this is the amount of time we
# wait before assuming that the server is unable to accept connections.
# increase the time
c.Spawner.http_timeout = 3000

# this is to submit the jupyter notebook job 
# configure according to your available number of nodes
# and the queue, I am using the CUDA (gpu)
c.TorqueSpawner.batch_script = '''
   #!/bin/sh
   #PBS -l nodes=1:CUDA:ppn=24
   #PBS -M <your email> 
   #PBS -m abe

   export CXX=/public/apps/gcc/4.4.7
   export PATH="/public/apps/cuda/7.0/bin:$PATH"
   export LD_LIBRARY_PATH="/usr/local/lib:/usr/lib:/home/mfahmed/anaconda3/lib:$PATH"
   export LD_LIBRARY_PATH=/public/apps/cuda/7.0/lib64:/public/apps/cuda/7.0/lib:$LD_LIBRARY_PATH
   export CUDA_ROOT=/public/apps/cuda/7.0
   export CUDA_LAUNCH_BLOCKING=0
   export JPY_API_TOKEN='<your generated api token>'
   cd $PBS_O_WORKDIR

   {cmd}
   '''
```
