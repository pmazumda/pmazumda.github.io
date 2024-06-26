--- 
layout: post 
title: Determine ulimit settings on Linux. 
date: '2016-07-13T15:31:00.002+05:30' 
author: Middlewarebytes
sitemap:
  lastmod: 2022-05-31
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags: 
modified_time: '2016-07-13T15:32:56.554+05:30'
blogger_orig_url: https://middlewarebytes.blogspot.com/2016/07/how-to-determine-ulimit-settings-of.html 
subtitle: Determine ulimit settings on Linux.
---

*Often there are situations arising where we need to determine the ulimit settings of a running application server process  like determining the maximum number of file descriptors for a given process or generating a system core from a process.*


In order to find out, we can follow the below steps I am listing down here
  
### Step 1: Determine the Process ID (PID) of the IBM WebSphere Application Server process to be investigated.


If the PID is already known, proceed to step 2.


If the PID is unknown then inspect the contents of the following file to determine the PID:


`<install_root>/profiles/<profile_name>/logs/<server_name>/<server_name>.pid`


This file should exist if the server is currently running. Exceptional circumstances could lead to a situation where this file exists and the server is not running.


The number contained within this file is the PID of the running server. We can even execute :- 


`ps -ef | grep java or ps -ef | grep <server_name>` which would give us the pid of the process.


### Step 2: Once the PID is known, inspect the file at the following location:

Location:     /proc/<PID>

The contents of this file is similar to the output of the __ulimit__ __-a__ command.This file will have a list of ulimit parameters and their associated values for the specified PID.

```

Limit                     Soft Limit  

Max cpu time              unlimited 

Max file size             unlimited 

Max data size             unlimited 

Max stack size            10485760  

Max core file size        unlimited 

Max processes             unlimited 

Max open files            8192   

```

### Limitations:

This method is only available to more recent versions of IBM WebSphere Application Server supported Linux operating systems.

There may be some incremental maintenance that can be applied to support this method prior to the above versions and maintenance levels as only major maintenance levels were tested.


### Reference: [How to determine ulimit settings](http://www-01.ibm.com/support/docview.wss?uid=swg21407889)

--------------------------------------------------------------------