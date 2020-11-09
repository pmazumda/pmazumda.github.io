---
layout: post
title: What is a HAProxy and how does it work ?
date: '2020-10-26T20:47:00.000+05:30'
author: Middlewarebytes
tags:
- WebServer
- Loadbalancer
- proxyserver
- reverseproxy
- TCP
- http
---


What is HAProxy ?

HAProxy (High Availability Proxy) is a TCP/HTTP load balancer and proxy server that allows a webserver to spread incoming requests across multiple endpoints.Recently it has become one of the commonly used open source load-balancer  for many of the  big websites around the world such as airbnb, reddit, tumblr and many others.It is particularly suited for high traffic sites,this is useful in cases where too many concurrent connections over-saturate the capability of a single server. Instead of a client connecting to a single server which processes all of the requests, the client will connect to an HAProxy instance, which will use a reverse proxy to forward the request to one of the available endpoints, based on a load-balancing algorithm.

Its mode of operation makes its integration into existing architectures very easy and riskless, while still offering the possibility not to expose fragile web servers to the net, such as below : 

![HAProxy](/img/postimages/haproxy-pmode.png?raw=true "HAProxy")

Initial Configuration

Review the default configuration file at /etc/haproxy/haproxy.cfg, which is created automatically during installation. This file defines a standard setup without any load balancing:




global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # Default ciphers to use on SSL-enabled listening sockets.
    # For more information, see ciphers(1SSL). This list is from:
    #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    # An alternative list with additional directives can be obtained from
    #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
    ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
    ssl-default-bind-options no-sslv3

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http
The global section defines system level parameters such as file locations and the user and group under which HAProxy is executed. In most cases you will not need to change anything in this section. The user haproxy and group haproxy are both created during installation.




The defaults section defines additional logging parameters and options related to timeouts and errors. By default, both normal and error messages will be logged.

If you wish to disable normal operation messages from being logged you can add the following after option dontlognull:

option dontlog-normal

You can also choose to have the error logs in a separate log file:

option log-separate-errors
Configure Load Balancing
When you configure load balancing using HAProxy, there are two types of nodes which need to be defined: frontend and backend. The frontend is the node by which HAProxy listens for connections. Backend nodes are those by which HAProxy can forward requests. A third node type, the stats node, can be used to monitor the load balancer and the other two nodes.

Open /etc/haproxy/haproxy.cfg in a text editor and append the configuration for the front end:


frontend haproxynode
    bind *:80
    mode http
    default_backend backendnodes


This configuration block specifies a frontend node named haproxynode, which is bound to all network interfaces on port 80. It will listen for HTTP connections (it is possible to use TCP mode for other purposes) and it will use the back end backend nodes.

Add the back end configuration:






backend backendnodes
    balance roundrobin
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    server node1 192.168.1.3:8080 check
    server node2 192.168.1.4:8080 check



This defines backendnodes and specifies several configuration options:

The balance setting specifies the load-balancing strategy. In this case, the roundrobin strategy is used. This strategy uses each server in turn but allows for weights to be assigned to each server: servers with higher weights are used more frequently. Other strategies include static-rr, which is similar to roundrobin but does not allow weights to be adjusted on the fly; and leastconn, which will forward requests to the server with the lowest number of connections.

The forwardfor option ensures the forwarded request includes the actual client IP address.

The first http-request line allows the forwarded request to include the port of the client HTTP request. The second adds the proto-header containing https if ssl_fc, a HAProxy system variable, returns true. This will be the case if the connection was first made via an SSL/TLS transport layer.

Option httpchk defines the check HAProxy uses to test if a web server is still valid for forwarding requests. If the server does not respond to the defined request it will not be used for load balancing until it passes the test.

The server lines define the actual server nodes and their IP addresses, to which IP addresses will be forwarded. The servers defined here are node1 and node2, each of which will use the health check you have defined.

Add the optional stats node to the configuration:

listen stats
    bind :32700
    stats enable
    stats uri /
    stats hide-version
    stats auth someuser:password

The HAProxy stats node will listen on port 32700 for connections and is configured to hide the version of HAProxy as well as to require a password login. Replace password with a more secure password. In addition, it is recommended to disable stats login in production.

Here is the complete configuration file after modifications:



global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

    # Default SSL material locations
    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    # Default ciphers to use on SSL-enabled listening sockets.
    # For more information, see ciphers(1SSL). This list is from:
    #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    # An alternative list with additional directives can be obtained from
    #  https://mozilla.github.io/server-side-tls/ssl-config-generator/?server=haproxy
    ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
    ssl-default-bind-options no-sslv3

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

frontend haproxynode
    bind *:80
    mode http
    default_backend backendnodes

backend backendnodes
    balance roundrobin
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1\r\nHost:localhost
    server node1 192.168.1.3:8080 check
    server node2 192.168.1.4:8080 check

listen stats
    bind :32700
    stats enable
    stats uri /
    stats hide-version
    stats auth someuser:password

Monitoring

Now, any incoming requests to the HAProxy node at IP address 203.0.113.2 will be forwarded to an internally networked node with an IP address of either 192.168.1.3 or 192.168.1.4. These backend nodes will serve the HTTP requests. If at any time either of these nodes fails the health check, they will not be used to serve any requests until they pass the test.
In order to view statistics and monitor the health of the nodes, navigate to the IP address or domain name of the frontend node in a web browser at the assigned port, e.g., http://203.0.113.2:32700. This will display statistics such as the number of times a request was forwarded to a particular node as well the number of current and previous sessions handled by the frontend node.

Security , Maintenance

Vulnerabilities are very rarely encountered in haproxy, and its architecture significantly limits their impact and often allows easy workarounds.
It is  generally suggested starting HAProxy as root because it can then jail itself in a chroot and drop all of its privileges before starting the instances. This is not possible if it is not started as root because only root can execute chroot()


OSI ( Open Systems Interconnection ) Model



Performance

HAProxy is an event-driven, non-blocking engine combining a very fast I/O layer
with a priority-based, multi-threaded scheduler. As it is designed with a data
forwarding goal in mind, its architecture is optimized to move data as fast as
possible with the least possible operations. It focuses on optimizing the CPU
cache's efficiency by sticking connections to the same CPU as long as possible.
As such it implements a layered model offering bypass mechanisms at each level
ensuring data doesn't reach higher levels unless needed. Most of the processing
is performed in the kernel, and HAProxy does its best to help the kernel do the
work as fast as possible by giving some hints or by avoiding certain operation
when it guesses they could be grouped later. As a result, typical figures show
15% of the processing time spent in HAProxy versus 85% in the kernel in TCP or
HTTP close mode, and about 30% for HAProxy versus 70% for the kernel in HTTP
keep-alive mode.

A single process can run many proxy instances; configurations as large as
300000 distinct proxies in a single process were reported to run fine. A single
core, single CPU setup is far more than enough for more than 99% users, and as
such, users of containers and virtual machines are encouraged to use the
absolute smallest images they can get to save on operational costs and simplify
troubleshooting. However the machine HAProxy runs on must never ever swap, and
its CPU must not be artificially throttled (sub-CPU allocation in hypervisors)
nor be shared with compute-intensive processes which would induce a very high
context-switch latency.






