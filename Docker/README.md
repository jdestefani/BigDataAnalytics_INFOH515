# [INFO-H515 - Big Data Scalable Analytics](https://uv.ulb.ac.be/course/view.php?id=85246)
# Docker container for the course and for Big Data Geospatial Analysis in Kafka & Spark Streaming (PySpark)

This Dockerfile sets up a complete streaming environment for experimenting with Kafka and Spark streaming (PySpark). It installs

* Kafka 2.12-2.3;
* Spark 2.4.5. 

It additionnally installs

* Anaconda distribution for Python 3.7;
* Jupyter notebook for Python; 
* Geopandas python package for geospatial data (http://geopandas.org/index.html).

## 0. Docker installation

Docker is a software container platform, which allows to isolate OS environments. Its main benefits over virtual machines are a reduced footprint (starting from 60MB for a Linux distribution - for example [ubuntu](https://hub.docker.com/_/ubuntu/)), and an easy way reconfigure an OS environment. Note: The Docker container for this course is 7.7GB, mostly due to the Anaconda (>1GB) and Spark (>1GB) distributions.

For more information on Docker, see: 

* https://www.docker.com/what-docker
* https://en.wikipedia.org/wiki/Docker_(software)

**To download and install, see**: (NB: install the Community Edition)

* https://docs.docker.com/engine/installation

**Get started Docker tutorial:**

* https://docs.docker.com/get-started

We **strongly** encourage you to follow the tutorial, and learn how to build, run, pull and push a container.

## 1. Get started

In order to avoid building the image from scratch, a prebuilt image is made available from DockerHub, see below.

### 1.1. Pull image

#### Pull image and create a folder for your Project

The image is called ```ulb_infoh515``` and is available from DockerHub (Note: image is 7.7GB, make sure you have a reasonably good Internet conection).

To install the image, use the standard ```docker pull``` command 

```
docker pull jdestefani/ulb_infoh515
```

Git clone the repository for the course

```
git clone https://github.com/jdestefani/BigDataAnalytics_INFOH515
```

Cd to the `BigDataAnalytics_INFOH515` folder

```
cd BigDataAnalytics_INFOH515
```

Finally, give recursive permission to all for writing to it (ease the sharing with Docker container)

```
chmod -R a+rwx .
```

(or the equivalent command for Windows).

The Docker container should now be able to read/write to your **host ```BigDataAnalytics_INFOH515``` folder**.

### 1.2. Start container

### 1.2.1. Start Docker container

#### Linux / Mac

From the ```BigDataAnalytics_INFOH515``` folder, start the container with

```
docker run -v `pwd`:/home/guest/host -p 8888:8888 -p 4040:4040 -p 23:22 -it jdestefani/ulb_infoh515 bash

```

#### Windows (PowerShell with Windows Linux Subsystem (WSL) installed)

```
$nixPath="$(wsl wslpath -a '$PWD')".substring(10); docker run -v ${nixPath}:/home/guest/shared -p 8888:8888 -p 4040:4040 -p 23:22 -it jdestefani/ulb_infoh515 bash

```

#### Windows (PowerShell without Windows Linux Subsystem (WSL) installed)

```
$nixPath = (($pwd.Path -replace "\\","/") -replace ":","").ToLower().Trim("/"); docker run -v ${nixPath}:/home/guest/shared -p 8888:8888 -p 4040:4040 -p 23:22 -it jdestefani/ulb_infoh515 bash

```


Notes

* -v is used to share folder (right permissions given above will allow your changes to be saved on your computer). The "-v pwd:/home/guest/host" shares the local folder (i.e. folder containing Dockerfile, ipynb files, etc...) on your computer; 
* -it starts the Docker container in interactive mode, so you can use the console and Bash;
* -p is for sharing ports between the container and the host. 8888 is the notebook port, and 4040 the Spark UI port.


SSH allows to get a onnection to the container

```
ssh -p 23 guest@containerIP
```

where 'containerIP' is the IP of th container (127.0.0.1 on Linux). Password is 'guest'.


### 1.2.2. Start Services 

Once run, you are logged in as root in the container. Run the startup_script.sh (in /usr/bin), with: 

```
kafka_startup_script.sh
```

to start:

* SSH server. You can connect to the container using user 'guest' and password 'guest'
* Zookeeper server
* Kafka server


### 1.2.3. Connect, open notebook and start streaming

Connect as user 'guest' and go to 'host' folder (shared with the host)

```
su guest
```

Start Jupyter notebook

```
notebook
```

and connect from your browser at port host:8888 (where 'host' is the IP for your host. If run locally on your computer, this should be 127.0.0.1 or 192.168.99.100, check Docker documentation)


#### Connect to Spark UI

It is available in your browser at port 4040


## 2. Container configuration details

The container is based on CentOS 6 Linux distribution. The main steps of the building process are

* Install some common Linux tools (wget, unzip, tar, ssh tools, ...), and Java (1.8)
* Create a guest user (UID important for sharing folders with host!, see below), and install Spark and sbt, Kafka, Anaconda and Jupyter notbooks for the guest user
* Go back to root user, and install startup script (for starting SSH), sentenv.sh script to set up environment variables (JAVA, Kafka, Spark, ...)and spark-default.conf 


### User UID

In the Dockerfile, the line

```
RUN useradd guest -u 1000
```

creates the user under which the container will be run as a guest user. The username is 'guest', with password 'guest', and the '-u' parameter sets the linux UID for that user.

In order to make sharing of folders easier between the container and your host, **make sure this UID matches your user UID on the host**. You can see what your host UID is with

```
echo $UID
```

## 3. Troubleshooting

#### Issue: 127.0.0.1 refused to connect

On some versions of Mac OSX and Windows, the [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/) is used instead of the native Docker Engine on the system.

In this case, instead of connecting to the IP address of the localhost (i.e. 127.0.0.1), one should connect to the IP address of the virtual machine where the Docker Engine is running (192.168.99.100, by default).

For example, to connect to the Jupyter Notebook, you should open your browser at 192.168.99.100:8888

#### Issue: Shared folder empty

The [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/) on Windows 10 Home shares by default only the primary hard drive (i.e. the one where Windows is installed -> C:).

In case your machine has more than one hard disk two solutions exists:
- Either the BigDataAnalytics_INFOH515 folder has to be located on the main hard drive (C:) 
- Or, a manual mountpoint to the second hard drive must be created in the configuration of VirtualBox (the provisioner running the virtual machine containing Docker Engine) as described [here](https://stackoverflow.com/questions/48828406/unable-to-share-volume-with-docker-toolbox-on-windows-10).

The same problem could appear if you are using [Docker on Windows](https://docs.docker.com/docker-for-windows/install/) (instead of the Docker Toolbox), a solution can be found [here](https://rominirani.com/docker-on-windows-mounting-host-directories-d96f3f056a2c).

