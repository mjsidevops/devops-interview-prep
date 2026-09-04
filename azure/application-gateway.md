
Application gateway:
 - Its a web application traffic load balancer.
 - Routes the traffic based on URL paths and Host headers
 - Application Gateway v2 is the latest version of Application Gateway
 - For example, you can route requests with /images in the URL to servers optimized for images, while routing /video requests to servers optimized for video content. This application layer routing gives you more control over how traffic flows to your applications.
<img width="720" height="441" alt="image" src="https://github.com/user-attachments/assets/663238f2-b3c9-4ac8-965a-77991c2ff5e8" />
 - Application Gateway operates at the application layer (OSI layer 7)
 - It provides features like SSL/TLS termination, autoscaling, zone redundancy, and integration with Web Application Firewall for security.
 - It provides Application Gateway Ingress Controller (AGIC) for Azure Kubernetes Service (AKS).

 - Components of App gateway:
 ```yaml
 Application Gateway
│
├── Frontend IP
│
├── Listener
│
├── Routing Rule
│
├── Backend Pool
│
├── Backend HTTP Settings
│
├── Health Probe
│
└── WAF
```

- Frontend IP:
   - It is the IP where the clients connect to the App gateway.
   - It can be Public or Private or both.

- Listener:
   - A listener is essentially the entry point that waits for incoming requests.
   - You can configure a listener with: Port, Protocol(Http or https), SSL, Hostname
 
- Routing rules:
   - Defines where the request should go.
   - Write the rules to route from which listener to which backend target.
   - Types:
       Basic      - route traffic without evaluating request attributes or matching conditions.
       Path based - Route based on URL paths
       Advanced   - Route based on header values, it contains host, origin, cookie etc

- Backend pool:
   - The backend pool contains the destinations to which Application Gateway sends traffic.
   - Backends can be VM, VMSS, IP Address, FQDN(private dns zone), App services

- Backend Settings
   - It defines how Application Gateway communicates with the backend.
   - You can configure things such as:
       Backend protocol        --> Http, https, TCP, TLS
       Backend port            --> Backend port to connect to.
       Cookie-based affinity   --> Its a sticky sessions, that makes sure requests from the same client session are sent to the same backend server
       Request timeout
       Connection draining     --> When a backend server is being removed or disabled, Application Gateway stops sending new connections to that server but allows existing connections to complete before the server is taken out of service.
       Dedicated backend connection --> A separate connection for each client to backend.

- Health probes:
   - Application Gateway continuously checks whether backend servers are healthy.
   - Application Gateway stops sending traffic to unhealthy backends.
 
- SSL/TLS Termination:
   - Application Gateway can terminate HTTPS connections.
   - It means, if client making HTTPS request to App gateway, it decrypts the request there and can send HTTP traffic to backend. Optionally can send HTTPS traffic too to the backend so it will be End-to-end TLS/SSL.
 
- WAF:
   - Web Application firewall
   - WAF helps protect web applications from common attacks.
   - SQL Injection, Cross-site scripting (XSS), Malicious requests, DDos protection


Azure Traffic Manager:
 - Azure Traffic Manager is a DNS-based traffic load balancer that distributes traffic to your public-facing applications across global Azure regions.



1. what is private link in Azure application gateway?

- Azure Application Gateway Private Link enables you to establish secure, private connections to your Application Gateway from workloads spanning across virtual networks (VNets) and subscriptions. This feature provides private connectivity without exposing traffic to the public internet.
- The Private Link configuration defines the infrastructure that enables connections from Private Endpoints to your Application Gateway.
- Application Gateway Private Link allows an Application Gateway frontend to be exposed privately through Azure Private Link. The Application Gateway acts as the service provider using a Private Link Service, while consumers connect from their VNet through a Private Endpoint. This allows private connectivity without exposing the Application Gateway to the public internet.

<img width="684" height="294" alt="image" src="https://github.com/user-attachments/assets/2ac72968-e1f9-46f0-9c4f-8a7c72b2a25b" />
<br><br>

2. What is Azure Front door?
   
Azure Front Door is an advanced content delivery network (CDN) for the cloud. It's designed to provide fast, reliable, and secure access to your applications' static and dynamic web content globally. By using Microsoft's extensive global edge network, Azure Front Door provides efficient content delivery through global and local points of presence (PoPs) strategically positioned close to both enterprise and consumer users.

```yaml
User
 |
 v
Front Door Edge
 |
 +---- Cached content?
 |        |
 |       YES
 |        |
 |        v
 |      User
 |
 NO
 |
 v
Origin
```

It is particularly useful when an application has origins deployed across multiple regions and we need a single global entry point.
<br><br>

2. Difference between Azure Front door vs Application gateway?
   
Application Gateway = regional Layer 7 traffic management inside/around a VNet.
Azure Front Door = global Layer 7 traffic management at Microsoft's edge.

Both are Layer 7 HTTP/HTTPS services, but their scope and purpose are different. Application Gateway is a regional application delivery service that is deployed in a VNet and is used for regional Layer 7 routing, TLS termination and WAF. Azure Front Door is a global application delivery service that operates through Microsoft's global edge network and is used for global traffic routing, edge caching, acceleration and cross-region failover. For a multiregion application, we can use Front Door as the global entry point and Application Gateway behind it for regional traffic management.
