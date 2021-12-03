---
layout: post
title: Change Docker Root directory
date: '2021-12-27T10:30:00.000+05:30'
author: Middlewarebytes
tags: Docker

---
### Issue ?

The default directory of docker is *(/var/lib/docker)* and since this directory contains all the images , volumes etc,to get the basic information about your Docker configuration, execute:

     $ docker info

    The out put will be as which contains  information about the storage driver and the  docker root directory.
     Storage Driver: overlay2
     Docker Root Dir: /var/lib/docker
	 
This dictory can become quite large in a very short period. With the recent versions of docker, you can set the value of the data-root parameter to any custom path
of your choice, however being humans, some of us doesn't likes changes and we keep the default ones.. :) Eventually this directory is gonna grow , offcourse not if you are 
not gonna do anything after bringing up the friendly hello-world docker  container. 


### Firstly, Stop the docker daemon.. 

If you are running docker as root, you need to prefix **sudo** with the command, otherwise not.

> `sudo systemctl docker status`

or 

> `sudo service docker status`


### What next ?

Next, we need to edit the service startup script to include our changes. The location of the service startup script can be found by running:

> `sudo systemctl docker status`

The output of the command would be like :
 
 ![docker command status](/img/postimages/dockerstatus.png)
 
 
Edit the file *(/lib/systemd/system/docker.service)* in your text editor.

Add -g *(/path/to/docker/)* at the end of ExecStart directive. The complete line should look like this.

> ExecStart=/usr/bin/dockerd -g *(/path/to/docker/)*

Execute the below command

> `sudo systemctl daemon-reload`
> `sudo systemctl restart docker`


Execute the command to check the new docker directory

> `docker info | grep "loop file\|Dir"`

 ![New root directory](/img/postimages/dockerrootdirectory.png)

### Summary 

Docker is an important part of many people’s environments and tooling. Sometimes, Docker feels a bit like magic by solving issues in a very smart way without telling the user how things are done behind the scenes. Still, Docker is a regular tool that stores its heavy parts in locations that can be opened and changed.
