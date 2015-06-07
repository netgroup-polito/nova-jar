# novajar
Jar Driver for OpenStack Nova (based on NovaDocker: https://github.com/stackforge/nova-docker)

## Installation & Configuration

###1. Configure OpenStack to enable docker

###2. Clone this repository

###3. Install the driver
You should then install the required modules.

```
cd src/novadocker/
python setup.py install
```

###4. Enable the driver in Nova's configuration
Nova needs to be configured to use the Docker virt driver.

Edit the configuration file /etc/nova/nova.conf according to the following options:

```
[DEFAULT]
compute_driver = novadocker.virt.docker.DockerDriver
```

Create the directory /etc/nova/rootwrap.d, if it does not already exist, and inside that directory create a file "docker.filters" with the following content:

```
# nova-rootwrap command filters for setting up network in the docker driver
# This file should be owned by (and only-writeable by) the root user

[Filters]
# nova/virt/docker/driver.py: 'ln', '-sf', '/var/run/netns/.*'
ln: CommandFilter, /bin/ln, root
```

###5. In addition put this specific settings

Edit the configuration file /etc/nova/nova.conf according to the following options:

```
[docker]
docker_image_name=sebymiano92/javaubuntu
docker_jar_path=/bin/new_folder
```

Where:

1. *docker_image_name* is the name of the container where the jar will be inserted
2. *docker_jar_path* is the location inside the Docker container where put the downloaded jar.

###6. Glance configuration

Glance needs to be configured to support the "jar" container format. It's important to leave the default ones in order to not break an existing glance install.

```
[DEFAULT]
container_formats = ami,ari,aki,bare,ovf,docker,jar
```

###7. Enable the ImagePropertiesFilter in the controller
Nova-scheduler is the module on the controller node that decides on which host to run the selected instance. There are some standard filters available in nova.schedule.filters.
We have to enable the ImagePropertiesFilter. 
It filters hosts based on the properties defined on the instance’s image. It passes hosts that can support the specified image properties contained in the instance. 
In particular it filters hosts based on the architecture, hypervisor type, and virtual machine mode specified in the instance.

```
scheduler_available_filters = nova.scheduler.filters.all_filters
scheduler_default_filters = RetryFilter, AvailabilityZoneFilter, RamFilter, ComputeFilter, ComputeCapabilitiesFilter, ImagePropertiesFilter
```

and when we create a new image in glance we should add the property

```
hypervisor_type=novajar
```

in order to let nova-scheduler to select the right compute node which implements the *novajar* driver

## How to use it

#####1. First of all push a new image to Glance:

```
glance image-create --is-public=True --container-format=jar --disk-format=raw --name bridge --file /home/controller/Desktop/bridge.jar
```

#####2. Update image properties

```
glance image-update bridge --property hypervisor_type=novajar
```

optionally we can put additional command line parameters to the jar with this property

```
glance image-update bridge --property os_command_line=eth0 eth1
```

#####3. After that, you can launch the image from the OpenStack dashboard and nova will schedule it in one of the compute nodes which implement the novajar driver.
