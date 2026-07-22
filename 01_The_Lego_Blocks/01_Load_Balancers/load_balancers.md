# System Design: Load Balancers

Load balancers are a fundamental piece of infrastructure that underpins scalable and reliable applications. Whether building web applications, APIs, or complex distributed systems, understanding how load balancers function is critical.

At its core, a load balancer acts as a **traffic director** for incoming requests. 

![Traffic Director](images/intro_traffic_director.png)

It is a hardware device or a software component that distributes network or application traffic across multiple servers.

![API Routing](images/intro_api_routing.png)

This ensures that no single server becomes overwhelmed when multiple clients make requests simultaneously.

![Handling Multiple Clients](images/intro_multiple_clients.png)

## Why do we need Load Balancers?

> [!NOTE]
> This traffic distribution isn't just about avoiding overloads; it's about laying the groundwork for a more robust and efficient system.

![Load Balancer Benefits](images/benefits_overview.png)

1. **Distribute Workload**: Prevents any single server from becoming a bottleneck, ensuring consistent performance.
2. **Dynamic Scaling**: Enables us to add or remove resources dynamically as demand shifts (e.g. during peaks and valleys of daily traffic).
3. **Reduce Latency**: By intelligently distributing requests, load balancers improve response times.
4. **Enhance Availability**: Provides redundancy and failover options. If a backend server or internal network experiences issues, the application remains accessible.

![Failover and Availability](images/benefits_failover.png)

---

## Types of Load Balancers (By Deployment)

There are three primary environments where load balancers operate.

![Deployment Types Overview](images/deployment_types_overview.png)

* **Hardware Load Balancers**: Dedicated physical appliances known for robust performance and stability. Designed for high-demand enterprise environments in dedicated data centers.
![Hardware Load Balancers in Datacenter](images/deployment_hardware_datacenter.png)

* **Software Load Balancers**: Run on commodity hardware, offering greater flexibility and cost-effectiveness. Suitable for a wider range of applications.
![Software Load Balancers](images/deployment_software.png)

* **Cloud-based Load Balancers**: Managed services offered by cloud providers like AWS, GCP, and Azure. This approach reduces operational overhead by shifting the management burden to the cloud provider.
![Cloud-Based Load Balancers](images/deployment_cloud.png)

---

## Types of Load Balancers (By Network Layer)

Load balancers are most commonly classified by the OSI Network Layer at which they operate.

![Network Layer Types Overview](images/network_layer_types_overview.png)

### Layer 4 Load Balancers
* **Operates at:** The Transport Layer.
* **Routing logic:** Makes routing decisions based strictly on IP addresses, ports, and TCP or UDP connections.
* **Pros:** Because they don't inspect the content of the traffic, Layer 4 balancers are exceptionally fast and efficient.
* **Use Case:** Ideal for TCP traffic and basic load balancing tasks where content-based routing isn't required.

> [!TIP]
> Use Layer 4 load balancers for pure speed and simplicity.

![Layer 4 OSI Model](images/network_layer_4_osi.png)
![Layer 4 Routing Example 1](images/network_layer_4_routing_1.png)
![Layer 4 Routing Example 2](images/network_layer_4_routing_2.png)

### Layer 7 Load Balancers
* **Operates at:** The Application Layer.
* **Routing logic:** Deals specifically with HTTP and HTTPS. Enables routing decisions based on the actual content of the traffic (HTTP headers, URLs, cookies, and other application-specific data).
* **Pros:** Allows for content-based routing (e.g., routing a user to a specific microservice based on their URL) and advanced features like **SSL Termination**.
* **Cons:** Gives you more control but requires more processing power to read the packets.

![Layer 7 OSI Model](images/network_layer_7_osi.png)
![Layer 7 Routing Example](images/network_layer_7_routing.png)

> [!IMPORTANT]
> **SSL Termination:** Layer 7 load balancers can handle encryption and decryption at the balancer itself, improving backend performance by offloading the cryptographic work and centralizing SSL certificate management.

![SSL Offloading](images/network_layer_7_ssl_offloading.png)

### Global Server Load Balancers (GSLB)
* **Operates at:** A global, geographic level.
* **Routing logic:** Distributes traffic across multiple geographic locations. Considers factors like user proximity to Data Centers and overall health of backend infrastructure across the globe.
* **Mechanism:** Uses DNS-based routing or anycast networking to direct users to the nearest available data center.
* **Use Case:** Essential for applications with a global user base that require low-latency access and high resilience against regional failovers.

![Global Server Load Balancing Brands](images/gslb_overview_brands.png)
![Global Map Distribution](images/gslb_map_distribution.png)
![Global Routing via IP](images/gslb_routing_ip.png)
![Global Client Connections](images/gslb_client_connections.png)

---

## Load Balancing Algorithms

How do load balancers actually distribute traffic? It depends on the chosen algorithm. Selecting the right one significantly impacts efficiency.

![Algorithms Overview](images/algorithms_overview.png)

### 1. Round Robin
The simplest method. It sequentially distributes requests across available servers, rotating through them in a continuous loop.
![Round Robin](images/algorithms_1_round_robin.png)

### 2. Sticky Round Robin (Sticky Sessions)
Ties a client to a specific server by creating a session ID (usually via a cookie or client IP). Once created, all requests from that client go to the same server. Helpful for applications relying on server-side session data, though it makes scaling more complex.
![Sticky Sessions Diagram](images/algorithms_2_sticky_sessions.png)

### 3. Weighted Round Robin
Assigns weights to each server, allowing the balancer to send a proportionally higher number of requests to more capable servers, and fewer to those with limited resources.
![Weighted Round Robin](images/algorithms_3_weighted_round_robin.png)

### 4. IP / URL Hashing
Uses a hash function to consistently route the same IP or URL to the same server. This stateless approach is particularly useful for caching static content.
![IP Hashing Formula](images/algorithms_4_ip_url_hash.png)

### 5. Least Connections
Directs traffic to the server with the fewest active connections at any given time, ensuring a more evenly distributed load.
![Least Connections Load Balancing](images/algorithms_5_least_connections.png)

### 6. Least Time
Routes requests to the fastest or most responsive server, ensuring a snappy user experience and reduced latency.
![Least Time Routing](images/algorithms_6_least_time.png)

---

## Key Metrics to Monitor

Load balancers provide vital metrics for monitoring system health and performance:

* **Traffic Metrics**: Request rates and total connections provide insight into traffic volumes.
* **Performance Metrics**: Response time, latency, and throughput help evaluate the end-user experience.
* **Health Metrics**: Server health checks and failure rates alert you to backend issues.
* **Error Metrics**: HTTP error rates and dropped connections identify potential connectivity problems.

![Monitoring Metrics](images/metrics_monitoring.png)

---
*Source: [ByteByteGo Load Balancers Video](https://youtu.be/LQuuoHTyYz8?si=HmniSxlrroZDaVYr)*
