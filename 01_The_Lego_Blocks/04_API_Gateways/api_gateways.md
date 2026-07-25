# 04 - API Gateways

An **API Gateway** is a single point of entry for all clients of an application. It sits between the clients and a collection of backend microservices, acting as a reverse proxy to manage and route all incoming traffic.

<p align="center"><img src="images/1.%20APIGW%20Overview.png" width="800" alt="API Gateway Overview" /></p>

An API gateway typically provides several highly critical functions, including:
* Authentication and security policy enforcement
* Load balancing and circuit breaking
* Protocol translation and service discovery
* Monitoring, logging, analytics, and billing
* Caching

---

## 1. The Typical Client Request Flow

Let’s examine a typical flow of a client request traveling through the API gateway and onto the backend service.

### Step 1: The Client Request
The client sends a request to the API gateway. This request is typically HTTP-based (such as REST, GraphQL, or other higher-level abstractions).

<p align="center"><img src="images/2.%20APIGW%20Client%20Flow.png" width="800" alt="API Gateway Client Flow" /></p>

### Steps 2 & 3: Validation and Basic Security
The API gateway first validates the HTTP request. It checks the caller’s IP address and HTTP headers against its predefined **allow-lists** and **deny-lists**. 

<p align="center"><img src="images/3.%20APIGW%20Validation.png" width="800" alt="API Gateway Validation" /></p>

It can also perform **basic rate limit checks** based on attributes like the IP address. For example, it could instantly reject requests from a single IP address that is flooding the system and exceeding a certain rate limit.

<p align="center"><img src="images/4.%20Rate%20Limit%20Check.png" width="800" alt="Rate Limit Check" /></p>

### Step 4: Authentication & Authorization
The API gateway passes the request to an Identity Provider (IdP) for authentication and authorization. Once validated, the API gateway receives an authenticated session back from the provider, which dictates the "scope" or permissions of what that specific request is allowed to do.

<p align="center"><img src="images/5.%20APIGW%20Auth.png" width="800" alt="API Gateway Authentication" /></p>

### Step 5: Advanced Rate Limiting
Once the user's identity is known, a **higher-level rate-limit check** is applied against their authenticated session. (For example, a free-tier user might be allowed 100 requests per minute, while a premium user gets 1000). If the session is over its tier limit, the request is rejected.

<p align="center"><img src="images/6.%20High%20Level%20Rate%20Limit.png" width="800" alt="High Level Rate Limit" /></p>

### Steps 6 & 7: Service Discovery
With the help of a Service Discovery component, the API gateway locates the appropriate backend microservice to handle the request based on path matching (e.g., routing `/users` to the User Service and `/orders` to the Order Service).

<p align="center"><img src="images/7.%20Service%20Dis%20&%20Dy%20Routing.png" width="800" alt="Service Discovery and Dynamic Routing" /></p>

### Step 8: Protocol Translation
The API gateway can transform the incoming HTTP request into a different protocol required by the backend service. For example, it might translate an incoming REST request into a **gRPC** call for internal backend communication. 

When the response comes back from the backend service, the API gateway translates it back to the public-facing HTTP protocol and returns it to the client.

<p align="center"><img src="images/8.%20Protocol%20Conversion.png" width="800" alt="Protocol Conversion" /></p>

---

## 2. Other Critical Services

A proper API gateway provides several other critical services to ensure the health of the entire system:
* **Circuit Breaking:** It protects backend services from being overloaded by "breaking the circuit" and returning errors early if a backend service is struggling.
* **Observability:** It provides centralized logging, monitoring, and analytics to give operators a clear view of system health and traffic patterns.

<p align="center"><img src="images/9.%20Critical%20Services.png" width="800" alt="Critical Services" /></p>

---

## Conclusion

An API gateway is a critical piece of modern distributed infrastructure. To improve availability and reduce latency, it is typically deployed across multiple geographic regions, positioning it as close to the global client base as possible.

<p align="center"><img src="images/10.%20Conclusion%20.png" width="800" alt="Conclusion" /></p>

*(Source: [ByteByteGo API Gateway Video](https://youtu.be/6ULyxuHKxg8?si=mMWzwr0U0yae6YTR))*
