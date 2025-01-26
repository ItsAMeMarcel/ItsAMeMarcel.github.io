---
layout: post
title:  "Load Balancing with HAProxy"
date:   2025-01-18 17:22:15 +0100
categories: loadbalancing HAProxy
---

This article is a follow-up to my previous article about **load balancing** with **NGINX**. The example code is also based on the micro-learning project mentioned earlier.

# HAProxy vs. NGINX

In one of my previous posts, I talked about a simple **load balancing** setup with **NGINX**. This article will shed more light on the usage of the **NGINX** alternative, **HAProxy**.

While **NGINX** was mainly developed as a high performance web server, **HAProxy**  focused primarily on **load balancing** and **traffic management**.

The following table shows the key aspects of **HAProxy** compared to **NGINX**.

| **Aspect**          | **HAProxy**                                                | **NGINX**                                                |
|----------------------|-----------------------------------------------------------|----------------------------------------------------------|
| **Purpose**          | Designed for **load balancing** and high-performance traffic management. | Initially a **web server**, later extended with reverse proxy and load balancing features. |
| **Performance**      | Optimized for pure load balancing; handles millions of concurrent connections efficiently. | High-performance, but slightly less efficient for large-scale traffic due to additional features (e.g., caching). |
| **Protocol Support** | Native support for **HTTP**, **HTTPS**, and **TCP**; limited support for **UDP**. Best for Layer 4 and Layer 7 balancing. | Supports **HTTP**, **HTTPS**, **TCP**, **UDP**, WebSockets, and more, offering broader versatility. |
| **SSL/TLS**          | Supports SSL/TLS termination but requires more manual configuration; limited automation for certificates. | Fully integrated SSL/TLS support with features like SSL offloading and automated certificate management (e.g., Let's Encrypt). |
| **Configuration**    | Simplified configuration for load balancing; focuses on traffic management, health checks, and connection handling. | Highly flexible for complex routing, URL rewriting, caching, and serving static/dynamic content alongside load balancing. |
| **Primary Use Cases**| Ideal for **high-performance load balancing** and advanced traffic management. | Best for setups needing a combination of **web server**, reverse proxy, and load balancer. |
| **Are They Competitors?** | Not entirely; HAProxy is better for pure load balancing, while NGINX excels as a web server with additional reverse proxy and load balancing capabilities. | Not entirely; NGINX is better suited for web server tasks with added proxy and caching functionality. |

# HAProxy Example

## Setup

The following example is based on the **NGINX** example. To set up the containers, we use *Podman*. Alternatively *Docker* can also be used. 

The project structure looks as follows:


```
.
├── app
│   ├── Dockerfile
│   └── index.php
├── compose.yaml
└── haproxy
    └── haproxy.cnf
```

As architecture, we use an **HAProxy** server that employs the **Round Robin** pattern to connect the frontend to three PHP servers that are providing a simple website. The website includes the hostname of the server we are connected to. With that, we can easily verify that our setup works.

The diagram below shows the basic architecture:

![HAProxy architecture](https://cdn.uploads.micro.blog/189931/2025/bp9-diagram.png)

## Configuration

A simple **Round Robin** configuration looks like this:

```
global
    log /dev/log local0
    log /dev/log local1 notice
    maxconn 200

defaults
    log     global
    option  httplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:8080
    mode http

    default_backend backend_servers

backend backend_servers
    mode http

    balance roundrobin
    server app1 app1:80 check
    server app2 app2:80 check
    server app3 app3:80 check

    http-request set-header X-Backend-Hostname %[src]
```

Below you can find a short explanation of each part of the configuration.

**Global Section**
- **log /dev/log local0 & local1 notice**: Logs to system log with specified facilities and priority.
- **maxconn 200**: Limits concurrent connections to 200.

**Defaults Section**
- **log global**: Uses global logging settings.
- **option httplog**: Enables detailed HTTP request logging.
- **timeout connect 5000ms**: 5-second timeout for connections.
- **timeout client/server 50000ms**: 50-second timeout for client/server interactions.

**frontend (`http_front`)**
- **bind :8080**: Listens on port 8080 for all IPs.
- **mode http**: Operates in HTTP mode.
- **default_backend backend_servers**: Routes traffic to the backend `backend_servers`.

**Backend (`backend_servers`)**
- **mode http**: Operates in HTTP mode.
- **balance roundrobin**: Distributes traffic evenly among servers.
- **server app1/app2/app3**: Defines backend servers on port 80 with health checks.
- **http-request set-header X-Backend-Hostname %[src]**: Adds client source address as a header (`X-Backend-Hostname`).

# Conclusion

After setting up **NGINX** and **HAProxy**, I conclude that, in this context, both **load balancers** perform well. The differences in the setup is quite small. I plan to continue testing more complex setups, such as those involving **SSL/TLS**, with both **load balancers**.

 