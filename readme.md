# Docker for Windows Quick Start Tutorial

#### Get Docker for Windows working and smoke test your .NET Framework applications


### Installing Docker for Windows
Use Windows 10 or Windows Server 2016.  Be sure to manually check for and apply all OS updates from Microsoft.  I ran my tests on Windows 10 Pro Fall Creators Update Version 1709 Build 16299.125  
```Ctrl+Esc, winver [ENTER]```

Docker is not compatible with Windows Server 2012 or below.

Docker uses Hyper-V to run the containers on Windows.  Your computer or VM must be capable of hardware virtualization.  Sometimes your BIOS will have a setting that needs configured to enable virtualization.  VMware VMs require that virtualization be enabled for the guest OS in the VM's CPU configuration.  

Your computer or VM should have 4 - 8 cores and 8 - 24GB RAM for initial testing.  VMware ESXi 6+ is required by Docker if using VMware virtual machines.

Once you have a Windows Server 2016 or Windows 10 instance working and on the network with internet access, download Docker for Windows from the Docker Store and install it.  

[Download Docker Community Edition](https://www.docker.com/community-edition)

* _Note: You can also install a native docker windows service instead of the Docker for Windows program.  I recommend saving that for a later session._

If virtualization is enabled and working on your computer, **Docker will install and start up in linux mode**.  Switch to Windows containers using the Docker system tray icon.  Right-click the Docker whale icon.

If you run into any blockers check out this document: 

https://docs.docker.com/docker-for-windows


### Working in Powershell

Open a powershell console or ISE as Administrator.


___Get Docker run info___
```powershell
docker info
```

___Enable Docker Syntax Tabbing in Powershell___
* _NOTE: This step is not required.  The purpose is to enable code completetion for Docker commands in Powershell._
```powershell
Set-Executionpolicy RemoteSigned
Install-Module posh-docker
Import-Module posh-docker
```

```powershell
docker [tab]
docker --help
```

### Image and Container Development and Deployment Workflow
* Download Docker image from an on-prem or cloud-based Docker repository 
* Build a new image that includes your application
* Build new containers from that image 
* Run containers
* Stop Containers
* Remove containers
* Monitor containers

Once images are pulled into Docker they are used to create custom images and/or create new containers (copied). A base image generally remains untouched and you can create as many different images and containers from it as needed. You can download ready-made images or you can make your own. For this excersize we will download a Windows image from the official Docker Respository automatically and customize it with our own application.  From that custom image we will create containers that serve the application.  

### Get Application Binaries
You can download a .NET Framework project from Github and try to build it but it is good idea to create a new project in Visual Studio or use an existing one.  Start simple, if your existing application has an extensive web.config you should create a new project for this test.  

If you are not a programmer, don't worry.  It is easy to create and build an example project in Visual Studio without knowing how to program.  By creating a new project, you can introduce external dependencies--such as a database--in stages to prevent obscurring the test with unneeded complexity.  You can find examples on the web for creating new projects in Visual Studio but you are probably clever enough to poke around and get it done.  

It is a good idea to instrument your application with **Azure Application Insights** at this point--if you have an Azure account.  This is easy to accomplish from within Visual Studio by right-clicking on the project.  It doesnt cost anything at this scale.  

### Prepare the application for Docker
Publish your application to a folder by right-clicking the project in Visual Studio.  Go to that folder and create a file named ``` Dockerfile```

Include this text in the ```Dockerfile```

```dockerfile
FROM microsoft/aspnet
COPY . /inetpub/wwwroot
```
The FROM command tells Docker what image to build your container from.  If the ```microsoft/aspnet``` base image is not in Docker already, Docker will pull it from the web.  The image is 7GB.  The COPY command sends all of the files from the current publish folder to the /inetpub/wwwroot folder on the new image. 

### Build a New Image
Open a powershell console in the published folder, and run this:

```
docker build -t myimage .
```

This command references the Dockerfile (by convention) and pulls down the ```microsoft/aspnet``` image from the official Docker repository and combines it with your application to create a new image called: myimage.  You should have two images now. Verify.

```
docker images
```
### Create and Run a New App Container
```
docker run -d --name mycontainer myimage
```

Your new container and application should now be live!

You can run the above command with different container names to create as many containers as you want for your application--until you run out of memory.  Of course this would provide no benefit outisde of development and testing.  You would want to start up containers on distinct resources or employ something like Docker Swarm or Kubernetes to provide for high availability and scalability. 

List the new container(s): ```docker ps```

### Test the App

Get the IP address of the app:

```
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" mycontainer
```

Plug that IP address into a browser to visit the site (or hit the API).

### Stopping and Removing Containers
```
docker stop mycontainer
docker rm mycontainer
```

###  Monitoring Docker and Containers

__View container stats and Docker logs from the command line___
```
docker stats --no-stream
docker stats --all --format "table {{.ID}}\t{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker logs mycontainer
```

__Interact with Docker and containers via the Docker API__

Enable **Expose daemon** in Docker General Settings to enable the API--via the tray icon right-click menu.  These links should work if you have enabled Docker Daemon access and are running Docker on your local machine with a live container named: mycontainer.

http://localhost:2375/v1.32/containers/json

http://localhost:2375/v1.32/containers/mycontainer/json

http://localhost:2375/v1.32/containers/mycontainer/stats

http://localhost:2375/v1.32/containers/mycontainer/stats?stream=false

See: [Docker API Reference](https://docs.docker.com/engine/api/v1.32/)

Extensive Docker Daemon logs are located in:
```C:\Users\username\AppData\Local\Docker```

__Kitematic__

Kitematic is a GUI alternative to the command line.  You have to download, unzip, and place the files for Kitematic in ```C:\Program Files\Docker\Kitematic```.  Then you can run it from the Docker tray menu.  It doesnt provide a stats API interface for some reason but otherwise makes for a nice, pretty place to accidentally break things. 
