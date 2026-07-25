# 02 - Caching

Caching is a common technique in modern computing to enhance system performance and reduce response time. From the front end to the back end, caching plays a crucial role in improving the efficiency of various applications and systems.

<p align="center"><img src="images/1.%20what%20is%20caching.png" width="800" alt="What is caching" /></p>

A typical system architecture involves several layers of caching. At each layer, there are multiple strategies and mechanisms for caching data, depending on the requirements and constraints of the specific application.

<p align="center"><img src="images/2.%20Layers%20of%20caching.png" width="800" alt="Layers of caching" /></p>

---

## 1. Hardware Caches

Before diving into a typical system architecture, let’s zoom in and look at how prevalent caching is within each computer itself.

Let’s first look at computer hardware. The most common hardware caches are L1, L2, and L3 caches.

<p align="center"><img src="images/3.%20Computer%20Caches.png" width="800" alt="Computer Caches" /></p>

* **L1 Cache:** The smallest and fastest cache, typically integrated into the CPU itself. It stores frequently accessed data and instructions, allowing the CPU to quickly access them without having to fetch them from slower memory.
* **L2 Cache:** Larger but slower than L1 cache, and is typically located on the CPU die or on a separate chip.
* **L3 Cache:** Even larger and slower than L2 cache, and is often shared between multiple CPU cores.

Another common hardware cache is the **Translation Lookaside Buffer (TLB)**. It stores recently used virtual-to-physical address translations. It is used by the CPU to quickly translate virtual memory addresses to physical memory addresses, reducing the time needed to access data from memory.

<p align="center"><img src="images/4.%20TLB.png" width="800" alt="TLB" /></p>

---

## 2. Operating System Caches

At the operating system level, there are page caches and other file system caches.

### Page Cache
The Page cache is managed by the operating system and resides in main memory. It is used to store recently used disk blocks in memory. When a program requests data from the disk, the operating system can quickly retrieve the data from memory instead of reading it from disk.

<p align="center"><img src="images/5.%20OS%20Page%20Cache.png" width="800" alt="OS Page Cache" /></p>

### Inode Cache
There are other caches managed by the operating system, such as the inode cache. These caches are used to speed up file system operations by reducing the number of disk accesses required to access files and directories.

<p align="center"><img src="images/6.%20%20OS%20Inode%20cache.png" width="800" alt="OS Inode Cache" /></p>

---

## 3. Application System Architecture Caching

Now let’s zoom out and look at how caching is used in a typical application system architecture.

### Client-Side / Browser Caching
On the application front end, web browsers can cache HTTP responses to enable faster retrieval of data. 

<p align="center"><img src="images/7.%20Client%20Side%20Cache%201.png" width="800" alt="Client Side Cache 1" /></p>

When we request data over HTTP for the first time, and it is returned with an expiration policy in the HTTP header; if we request the same data again, the browser returns the data from its cache if available.

<p align="center"><img src="images/8.%20Client%20Side%20Cache%202.png" width="800" alt="Client Side Cache 2" /></p>

### Content Delivery Networks (CDNs)
Content Delivery Networks (CDNs) are widely used to improve the delivery of static content, such as images, videos, and other web assets. One of the ways that CDNs speeds up content delivery is through caching.

<p align="center"><img src="images/9.%20CDN%20Caching%201.png" width="800" alt="CDN Caching 1" /></p>

When a user requests content from a CDN, the CDN network looks for the requested content in its cache.

<p align="center"><img src="images/10.%20CDN%20Caching%202.png" width="800" alt="CDN Caching 2" /></p>

If the content is not already in the cache, the CDN fetches it from the origin server and caches it on its edge servers. When another user requests the same content, the CDN can deliver the content directly from its cache, eliminating the need to fetch it from the origin server again.

<p align="center"><img src="images/11.%20CDN%20Caching%203.png" width="800" alt="CDN Caching 3" /></p>

### Load Balancer Caching
Some load balancers can cache resources to reduce the load on back-end servers. When a user requests content from a server behind a load balancer, the load balancer can cache the response and serve it directly to future users who request the same content. This can improve response times and reduce the load on back-end servers.

<p align="center"><img src="images/12.%20LB%20Caching.png" width="800" alt="LB Caching" /></p>

### Messaging Infrastructure
Caching does not always have to be in memory. In the messaging infrastructure, message brokers such as Kafka can cache a massive amount of messages on disk. This allows consumers to retrieve the messages at their own pace. The messages can be cached for a long period of time based on the retention policy.

<p align="center"><img src="images/13.%20Kafka%20Caching.png" width="800" alt="Kafka Caching" /></p>

### Distributed Caches
Distributed caches such as Redis can store key-value pairs in memory, providing high read/write performance compared to traditional databases.

<p align="center"><img src="images/14.%20Redis%20caching.png" width="800" alt="Redis Caching" /></p>

### Search Engines
Full-text search engines like Elastic Search can index data for document search and log search, providing quick and efficient access to specific data.

<p align="center"><img src="images/15.%20Elastic%20Caching.png" width="800" alt="Elastic Caching" /></p>

### Database Caching
Even within the database, there are multiple levels of caching available. 
* Data is typically written to a write-ahead log (WAL) before being indexed in a B-tree.
* The **buffer pool** is a memory area used to cache query results.
* **Materialized views** can precompute query results for faster performance.
* The **transaction log** records all transactions and updates to the database, while the **replication log** tracks the replication state in a database cluster.

<p align="center"><img src="images/16.%20RDBMS%20Cache.png" width="800" alt="RDBMS Cache" /></p>

---

## Conclusion
Overall, caching data is an essential technique for optimizing system performance and reducing response time. From the front end to the back end, there are many layers of caching to improve the efficiency of various applications and systems.

*(Source: [ByteByteGo Caching Video](https://youtu.be/dGAgxozNWFE?si=sH9B-VvsPdHxxkB_))*
