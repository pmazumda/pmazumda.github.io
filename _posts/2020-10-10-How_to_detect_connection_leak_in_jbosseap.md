---
layout: post
title: How to identify a connection in Jboss Enterprise Application Server ?
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


How to detect leaked data-source connections in Red Hat JBoss Enterprise Application Platform (EAP) – 6 / 7 

###Issue
•	How can I detect leaked datasource connections?
•	How can I identify code which leaks datasource connections?
•	"IJ000453: Unable to get managed connection for ..."
•	"IJ000655: No managed connections available within configured blocking timeout ..."
•	Application is not able to obtain a new connection from a connection pool but INACTIVE session count (reported by the database server) is increasing.
•	Does JBoss close a connection leak automatically?
###Resolution
To enable the cached connection manager (CCM) to identify a connection leak:
•	Enable the CCM for the datasource. It defaults to true if it is not explicitly specified but you may set use-ccm="true" explicitly.
    <subsystem xmlns="urn:jboss:domain:datasources:1.1">
       <datasources>
          <datasource ... enabled="true" use-ccm="true">
             ...
          </datasource>
       </datasources>
    </subsystem>
•	Verify that <cached-connection-manager> exists in the jca subsystem and set debug="true".
       <subsystem xmlns="urn:jboss:domain:jca:1.1">
          ...
          <cached-connection-manager debug="true" error="false"/>
          ...
       </subsystem>
o	CLI commands
	NOTE : while CLI may be run to configure running servers, a restart should be performed for the server where debug is required to ensure the debug setting is applied to the connection pool
	Standalone Mode
/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=debug,value=true)
	Domain Mode
/profile=<your_profile_here>/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=debug,value=true)`
o	Setting debug="true" will:
	Log an INFO message indicating that JBoss is "Closing a connection for you. Please close them yourself"
	Generate a stacktrace for the code where the leaked connection was first opened.
	Close the leaked connection
•	The additional property error may be used to raise a RuntimeException and generate an ERROR message in the log when it is set to true.
•	NOTE : For EAP releases earlier than EAP 7.1, jta MUST also be set to true for the datasource in order for cached connection manager debug to work.
•	Restart JBoss to ensure that the changes are applied.
Examples
Example with debug="true" without error="true" (i.e. <cached-connection-manager debug="true" /> or <cached-connection-manager debug="true" error="false" />):
INFO  [org.jboss.jca.core.api.connectionmanager.ccm.CachedConnectionManager] (http-/127.0.0.1:8080-1) IJ000100: Closing a connection for you. Please close them yourself: org.jboss.jca.adapters.jdbc.jdk6.WrappedConnectionJDK6@6f1170a9: java.lang.Throwable: STACKTRACE
    at org.jboss.jca.core.connectionmanager.ccm.CachedConnectionManagerImpl.registerConnection(CachedConnectionManagerImpl.java:269)
    at org.jboss.jca.core.connectionmanager.AbstractConnectionManager.allocateConnection(AbstractConnectionManager.java:495)
    at org.jboss.jca.adapters.jdbc.WrapperDataSource.getConnection(WrapperDataSource.java:139)
        ...
Example with debug="true" and error="true" (i.e. <cached-connection-manager debug="true" error="true"/>):
INFO  [org.jboss.jca.core.api.connectionmanager.ccm.CachedConnectionManager] (http-/127.0.0.1:8080-1) IJ000100: Closing a connection for you. Please close them yourself: org.jboss.jca.adapters.jdbc.jdk6.WrappedConnectionJDK6@27c94e11: java.lang.Throwable: STACKTRACE
    at org.jboss.jca.core.connectionmanager.ccm.CachedConnectionManagerImpl.registerConnection(CachedConnectionManagerImpl.java:269)
    at org.jboss.jca.core.connectionmanager.AbstractConnectionManager.allocateConnection(AbstractConnectionManager.java:495)
    at org.jboss.jca.adapters.jdbc.WrapperDataSource.getConnection(WrapperDataSource.java:139)
        ...
ERROR [org.apache.catalina.connector.CoyoteAdapter] (http-/127.0.0.1:8080-1) An exception or error occurred in the container during the request processing: java.lang.RuntimeException: java.lang.RuntimeException: javax.resource.ResourceException: IJ000151: Some connections were not closed, see the log for the allocation stacktraces
    at org.jboss.as.web.ThreadSetupBindingListener.unbind(ThreadSetupBindingListener.java:67) [jboss-as-web-7.1.3.Final-redhat-4.jar:7.1.3.Final-redhat-4]
        ...
Caused by: java.lang.RuntimeException: javax.resource.ResourceException: IJ000151: Some connections were not closed, see the log for the allocation stacktraces
    at org.jboss.as.connector.deployers.ra.processors.CachedConnectionManagerSetupProcessor$CachedConnectionManagerSetupAction.teardown(CachedConnectionManagerSetupProcessor.java:85)
    at org.jboss.as.web.ThreadSetupBindingListener.unbind(ThreadSetupBindingListener.java:61) [jboss-as-web-7.1.3.Final-redhat-4.jar:7.1.3.Final-redhat-4]
    ... 8 more
Caused by: javax.resource.ResourceException: IJ000151: Some connections were not closed, see the log for the allocation stacktraces
    at org.jboss.jca.core.connectionmanager.ccm.CachedConnectionManagerImpl.popMetaAwareObject(CachedConnectionManagerImpl.java:249)
    at org.jboss.as.connector.deployers.ra.processors.CachedConnectionManagerSetupProcessor$CachedConnectionManagerSetupAction.teardown(CachedConnectionManagerSetupProcessor.java:83)
    ... 9 more
Additional Notes
•	If leaks are believed to be present but nothing is reported by the CCM verify the following:
o	The datasource must not be configured with use-ccm="false"
o	The datasource must not be configured with jta="false".
	In EAP 7.1 and later, the limitation of non-JTA pools that prevented use of cached connection manager debug has been removed. However, in most cases, JTA should not be disabled for production datasources.
	See this article for details on appropriate jta configuration for your datasources.
	In the EAP 6.4 release, a new connection pool implementation called LeakDumperManagedConnectionPool has been added. See Using the LeakDumperManagedConnectionPool for additional details. This is especially useful if you must use datasource pools for which the jta property cannot be set to true.
o	The minimum logging level must be INFO for org.jboss.jca
•	Activating debug will have some impact on performance and log file size and it is, therefore, recommended that the debug configuration be used only during development or test cycles.
o	See also What causes 'Closing a connection for you... in EAP 6 which demonstrates proper closure of connection resources.
o	After verifying that no leaks exist/remain, restore the configuration by removing the debug="true" setting or using <cached-connection-manager debug="false"/> (before deploying in production).
o	The debug feature is not intended to be used as a resolution for connections leaked by application code.
•	To track the usage of datasource connections you can also enable datasource logging
•	Setting error to true is not recommended for production deployment. Where debug is required in production deployments, setting debug="true" should provide adequate information to diagnose the issue.
•	In addition to providing support for debugging, the CCM is also used to support lazy enlistment
o	The CCM (without debug enabled) is enabled by default and its use is normal for production systems

