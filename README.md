# MustHowTo

This is a repository for getting started with deploying applications to the Must datacentre.
My examples focus on my use case, AI training.
You can take a look at the `examples` folder for a quick start.

## The Tech Stack

### Apptainer

Apptainer needs to be installed, as you will need to build your image on your machine.
Documentation is available [here](https://apptainer.org/docs/user/main/build_a_container.html).

Apptainer is a container platform designed for HPC environments.
It can bootstrap from Docker images and uses a Linux distribution as a base, allowing you to use apt commands if needed.
You should always build your images on your machine (and test them), then transfer them to the remote server.
An example file looks like this:
```
Bootstrap: docker
From: <your_docker_image>

%post

# Commands to execute after the image was pulled. Usually pip commands for python images, but apt commands are also possible.

%environment

# Your environment variables
```
After defining your .def file, you can build your image using this command: <br>
`apptainer build <image_name>.sif <def_filename>.def`

After doing this, you can transfer your image by using scp like so: <br>
`scp <abs_path_of_file_to_transfer> <your-username>@<your_server>.in2p3.fr:/<path>/<filename>` 

### HTCondor
Must uses HTCondor, to generate jobs. It is already present on the remote server, but for testing purposes you can install it.
The most common commands you will need to use are the following:
- `condor_q`: This one lets you see the status of your jobs, and of the other jobs, as to know the workload on the server.
- `condor_submit <your_submit_file>`: This one lets you submit a job to the queue

A submit file is composed of different fields you can fill. The first is the executable you want to use. After, you can specify a universe, usually vanilla, then the output files, and finally request the resources you need before queuing.
Here is a minimal example submit_file :
```
executable=/usr/bin/apptainer
universe=vanilla

output=results.output.$(Process)
error=results.error.$(Process)
log=results.log

request_cpus = 1
request_memory = 1GB
request_disk = 1GB

queue
```

Some specifications are possible like `request_gpus` or `+wantGpuType` for launching with a gpu for example.
See [this](https://doc.must-datacentre.fr/batch/gpu/) for more details on more specifications.