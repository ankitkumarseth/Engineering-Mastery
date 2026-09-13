# Handling Overload

## Academic Definition
In distributed systems, **Overload** occurs when the rate of incoming work exceeds the system's capacity to process it. Because no system has infinite capacity, handling overload gracefully is required to prevent total system collapse. **Graceful Degradation** is the architectural principle of deliberately sacrificing some non-critical functionality or dropping a fraction of traffic to keep the core system operational.

## Load Shedding vs Rate Limiting
* **Rate Limiting (Throttling):** Protects the system from a *specific user or client* consuming too many resources (e.g., 100 requests/min per user). Usually implemented at the API Gateway using algorithms like Token Bucket or Leaky Bucket.
* **Load Shedding:** Protects the *server itself* regardless of who the user is. If the server is at 95% overall capacity, it proactively starts dropping new requests (returning 503 Service Unavailable) to ensure the currently processing requests can finish successfully.

## Retries and Backoff
When a client experiences an error (or a load shedding 503), it typically retries the request. 
* **Exponential Backoff:** Waiting progressively longer between retries (e.g., 1s, 2s, 4s, 8s) to avoid overwhelming the struggling server.
* **Jitter:** Adding a random delay offset to the backoff time. Without Jitter, 1,000 clients might fail at exactly the same time, wait exactly 2 seconds, and then hit the server simultaneously again, creating a "Thundering Herd."

## Graceful Degradation in Practice
When overloaded or missing upstream dependencies, the system should return a degraded but functional response rather than failing completely.
* *Example:* Netflix failing to load personalized recommendations for a user, so it gracefully falls back to returning the hardcoded "Trending Now" global list instead of throwing a 500 error page.

## Principal Engineer Interview Context
Always talk about **Load Shedding** when discussing sudden traffic spikes. Mention that dropping traffic cleanly and fast is much better than the server running out of memory, entering a Garbage Collection (GC) death spiral, and serving *zero* traffic.
