#Modified batchspawner for Jupyterhub (UofM) [![Build Status](https://travis-ci.org/jupyterhub/batchspawner.svg?branch=master)](https://travis-ci.org/jupyterhub/batchspawner)
This is a custom spawner for Jupyterhub. This version of batchspawner is specificly modified to work with [University of Memphis](http://www.memphis.edu/hpc/configuration.php) HPC environment.

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
1. install jupyterhub from [this link](https://github.com/jdfreder/multiuser-server/blob/master/docs/getting-started.md). The project home is [here](https://github.com/jupyterhub/jupyterhub) 
2. generate jupyterhub configuration file (default configuration file name is jupyterhub_config.py)
   
   ```bash 
   $ jupyterhub --generate-config
   ```

3. get the package *batchspawner* from github 
   ```bash 
   $ git clone https://github.com/farukahmedatgithub/batchspawner.git
   ```

4. from root directory of this *batchspawner* (where setup.py is), run 
   ```bash 
   $ pip install -e  .
   ```
   If you do not actually need an editable version, you can simply run 
      
   ```bash
   $ pip install  https://github.com/farukahmedatgithub/batchspawner.git      (it did not work for me)
   ```
5. add or enable lines in jupyterhub_config.py for the spawner you intend to use, e.g.
   
   ```python
      c.JupyterHub.spawner_class = 'batchspawner.TorqueSpawner'
   ```

6. Depending on the spawner, additional configuration will likely be needed.


Here is the minimum example configuration given. You can copy and save it as jupyterhub_config.py file. For other configuratin parameters it will use the defaults. 

### Example configuration

```python

# Set the log level by value or name.
c.Application.log_level = 'DEBUG' # 0,10,20.... 


# The ip for this process
c.JupyterHub.hub_ip = 'xx.xx.xx.xx' # put your login node ip which is facing towards the computing node

#
######################################################################################################
#
# you may generate a longer random string and use as proxy_auth_token. You can put that in the following
# configuratino parameter or export to the CONFIGPROXY_AUTH_TOKEN env variable. 
# or use this command 
# $ export CONFIGPROXY_AUTH_TOKEN=`openssl rand -hex 32`
######################################################################################################
#

# Loaded from the CONFIGPROXY_AUTH_TOKEN env variable by default.
c.JupyterHub.proxy_auth_token = '<you put your auth token>'

# The class to use for spawning single-user servers.
# 
# Should be a subclass of Spawner.
c.JupyterHub.spawner_class = 'batchspawner.TorqueSpawner'


######################################################################################################
#
# to generate SSL certificate you may use the following command 
# $ openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
#
######################################################################################################


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

#
########################################################################################################
#
# to generate API token (for JPY_API_TOKEN) use this command from the same directory where you have the configuratin file
# $ jupyterhub token <your hpc username> 
#
# to export this API token use 
# $ export JPY_COOKIE_SECRET=`openssl rand -hex 1024`
########################################################################################################
#
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
   export JPY_API_TOKEN='<your generated api token, instructin below>'
   cd $PBS_O_WORKDIR

   {cmd}
   '''
```

The configuration is done. Now you have to login to the login-node with the gui enabled secure shell. For mac [XQuartz](https://www.xquartz.org/), windows [MobaXterm] can be used. How to guide can be found [here](https://uisapp2.iu.edu/confluence-prd/pages/viewpage.action?pageId=280461906). For the UofM HPC use

```bash
$ ssh -X <user name>@login1.memphis.edu
```

then open a browser in backend 

```bash
$ firefox &
```

Run the jupyterhub using following command 
```bash
$ jupyterhub --config <path to config file/jupyterhub_config.py>
```

At this stage open https://localhost:8000/ (note https://) at the browser which you just ran in background (firefox in this example). Login with your HPC username and the password. You are ready to go !!





