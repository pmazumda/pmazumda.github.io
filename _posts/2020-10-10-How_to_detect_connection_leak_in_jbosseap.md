---
layout: post
title: How to identify a connection leak in Jboss Enterprise Application Server(6/7)?
date: '2020-10-28T10:47:00.000+05:30'
author: Middlewarebytes
tags:
- Application Server
- JBoss
- Jboss EAP
- connection leak
- datasource
- oracle
---

Have you came accross issues with your application which is deployed in Jboss EAP trying to find answers for the following questions.

 - How can I detect leaked datasource connections?

 - How can I identify code which leaks datasource connections?

**OR** 

if the server log contains error messages with error codes  **IJ000453** , **IJ000655** it means that there is a connection leak at the code, this can give rise to problems such as application not able to fetch any free connections  from the pool whereas the **INACTIVE** connections in the database keeps increasing.


### Resolution

In order to identify the source of the connection leak, we need to enable the cached connection manager (CCM).

You can follow the below steps on how to  enable this:

    Enable the CCM for the datasource. It defaults to **true** if it is not explicitly specified but you may set **use-ccm="true"** explicitly.
    

	<subsystem xmlns="urn:jboss:domain:datasources:1.1">
       <datasources>
          <datasource ... enabled="true" use-ccm="true">
             ...
          </datasource>
       </datasources>
    </subsystem>
	
	
	Verify that <cached-connection-manager> exists in the jca subsystem and set **debug="true".**
	
	
    <subsystem xmlns="urn:jboss:domain:jca:1.1">
          ...
        <cached-connection-manager debug="true" error="false"/>
          ...
    </subsystem>
	   
	   
The same can be also implemented using  the CLI tool, if you are running the commands onto a running server, please note that these attributes are not dynamic in nature and needs a server restart in order to be applied to the connection pool.

#### CLI commands


- Standalone Mode

    - /subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=debug,value=true)
	
	
- Domain Mode

    - /profile=<your_profile_here>/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=debug,value=true)`

    - Setting debug="true" will:
        - Log an INFO message indicating that JBoss is "Closing a connection for you. Please close them yourself"
        - Generate a stacktrace for the code where the leaked connection was first opened.
	    - Close the leaked connection
        - The additional property error may be used to raise a RuntimeException and generate an ERROR message in the log when it is set to    true.


 - **NOTE:** Set jta also to **true** for the datasources in order for cached connection manager debug to work in Jboss EAP releases earlier than 7.1.


### Additional Notes

- If leaks are believed to be present but nothing is reported by the CCM verify the following:
    - The datasource must not be configured with **use-ccm="false"**
	- The datasource must not be configured with **jta="false".**
        - In EAP 7.1 and later, the limitation of non-JTA pools that prevented use of cached connection manager debug has been removed. However, in most cases, JTA should not be disabled for production datasources.
		- See this article for details on appropriate jta configuration for your datasources.
		- In the EAP 6.4 release, a new connection pool implementation called LeakDumperManagedConnectionPool has been added. See Using the LeakDumperManagedConnectionPool for additional details. This is especially useful if you must use datasource pools for which the jta property cannot be set to true
	- The minimum logging level must be INFO for _org.jboss.jca_


