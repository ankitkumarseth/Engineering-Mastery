# The 4 Golden Signals (Google SRE)

## Academic Definition
The four golden signals of monitoring are Latency, Traffic, Errors, and Saturation. According to Google SRE principles, if you can only measure four metrics of your user-facing system, focus on these four. They provide the most comprehensive, high-level view of system health.

## 1. Latency
The time it takes to service a request.
* **Crucial Detail:** Separate the latency of *successful* requests from *failed* requests. A failed request might fail instantly (e.g., connection refused or early timeout), making your average latency look artificially good.
* **Best Practice:** Use Percentiles (p95, p99) instead of Averages/Means. Averages hide the long tail of slow requests that impact the worst-case user experience.

## 2. Traffic
A measure of how much demand is being placed on your system.
* Web APIs: HTTP requests per second.
* Audio/Video: Network I/O rate.
* Key-Value Store: Transactions per second.

## 3. Errors
The rate of requests that fail, either explicitly, implicitly, or by policy.
* **Explicit Errors:** HTTP 500s.
* **Implicit Errors:** A 200 OK response that returns the wrong content or a truncated response.
* **Policy Errors:** A request that succeeds but takes 2 seconds when the SLO mandates < 500ms.

## 4. Saturation
How "full" your service is. A measure of your system fraction, emphasizing the resources that are most constrained (e.g., CPU, Memory, I/O bandwidth).
* **Crucial Detail:** Saturation is often a leading indicator of trouble. If a database CPU hits 80%, latency usually starts degrading significantly before it hits 100%.
* **Queue Depth:** In asynchronous systems, the size of the pending job queue is often the most accurate indicator of saturation.

## Principal Engineer Interview Context
When designing a system, explicitly mention setting up dashboards and alerting on these 4 signals *first*. "Before launching this microservice, I would instrument Prometheus to track the 4 golden signals, specifically setting latency alerts on the p99 threshold."
