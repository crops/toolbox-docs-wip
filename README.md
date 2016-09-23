## Introduction
This document explains how to use docker containers created to run Yocto Project tools in a Windows or Mac environment.
## Configure Containers
### Install docker toolbox
Follow instructions <https://www.docker.com/products/docker-toolbox>
### Change default vm settings
The default Virtual Box VM does not provide enough resources to give a good experience when building Yocto images. We recommend you create a new VM with at least 2 CPUs and 4GB of memory.
* Run **Docker Quickstart Terminal** and then run the following commands in that terminal.
* Remove the default vm

     ```
     docker-machine rm default
     ```

* Re-create the default vm
   * Choose the number of cpus with  **--virtualbox-cpu-count**. For this example we'll use two.
   * Choose the amount of RAM: **--virtualbox-memory**. This is also based on the host hardware. However, choose at least 4GB.
   * Choose the amount of disk space: **--virtualbox-disk-size**. It is recommended that this be at least 50GB since building generates a lot* of output. In this example we'll choose 50GB.
   * Create vm with new settings

    ```
    docker-machine create -d virtualbox --virtualbox-cpu-count=4 --virtualbox-memory=8192 --virtualbox-disk-size=50000 default
    ````

* Restart docker

    ```
    docker-machine stop
    exit
    ```

Then start open a new Docker Quickstart Terminal.

### Create the samba container
* In the new quickstart terminal create a volume as follows.

    ```
    docker volume create --name myvolume
    docker run -it --rm -v myvolume:/workdir busybox chown -R 1000:1000 /workdir
    ```

* Create a samba container that will allow you to see the files in the volume.

    ```
    docker create -t -p 137-139:137-139 -p 445:445 --name samba -v myvolume:/workdir crops/samba
    ```


* Start the samba container

    ```
    docker start samba
    ```

* Get the ip address used to talk to samba and open /workdir

    ```
    docker-machine ip
    ```

    The result of this command is the address you will use to see the workdir.
    In this example let's say the command returned 192.168.99.100. Now, to see
    the *workdir* open the file browser in windows (win+e) and type
    ```
    \\192.168.99.100\workdir
    ```

## Using the poky container
Before using the poky container, make sure the samba container is running. Note that if you have started it in a previous terminal it will still be running. Run poky container as follows,  note that we use the volume created above when specifying the workdir.

```
docker run --rm -it -v myvolume:/workdir crops/poky --workdir=/workdir
```

You will see a prompt that looks like

`pokyuser@892e5d2574d6:/workdir$`

Although this is called the poky container, it does not include the bitbake meta-data for the Yocto Project poky distro, instead it is a Linux environment with all dependencies already installed. See the [Yocto Quick Start Guide](http://www.yoctoproject.org/docs/current/yocto-project-qs/yocto-project-qs.html#releases) to get started with with Yocto project.

## Troubleshooting
### Cannot connect to docker

You may see the following error when starting the QuickStart terminal

	This machine has been allocated an IP address, but Docker Machine could not	reach it successfully.

	SSH for the machine should still work, but connecting to exposed ports, such as
	the Docker daemon port (usually <ip>:2376), may not work properly.

	You may need to add the route manually, or use another related workaround.

	This could be due to a VPN, proxy, or host file configuration issue.

First look in at Network Adapters section in Device Manager. If you see entries named "Virtual Box Bridged Network Driver" wit ha yellow bang against them, you should remove them. Typically simply uninstalling does not work do you must do the following instead

* Select driver entry
* Right click on it and select "Update Driver Sofware"
* In resulting window, click on "Browse my computer...."
* Then click on "Let me pick"
* Selected chosen driver and click on "Next"

If that does not fix the problem, start Oracle VirtualBox and remove host-only networks

* File -> Preferences -> Network -> Host only Network
* Select each network and hit '-' button
