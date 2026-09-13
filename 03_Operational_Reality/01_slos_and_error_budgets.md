# Service Level Objectives (SLOs) & Error Budgets

## Academic Definitions
From the Google SRE Book, the foundational terms are defined as follows:
* **SLI (Service Level Indicator):** A carefully defined quantitative measure of some aspect of the level of service that is provided. E.g., The proportion of HTTP GET requests to `/api/data` that have a 2XX status and complete in < 100ms.
* **SLO (Service Level Objective):** A target value or range of values for a service level that is measured by an SLI. E.g., The SLI will meet the target 99.9% of the time over a rolling 30-day window.
* **SLA (Service Level Agreement):** An explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs they contain. (Usually financial penalties).

### Key Rule for SLIs
**Measure from the user's perspective.** If your server says 100% uptime but the load balancer is dropping 50% of traffic, the user experiences 50% availability.

## Error Budgets
An error budget is simply `100% - SLO`. For a 99.9% SLO, you have an error budget of 0.1% of requests.
* **The Goal:** It aligns Product and Engineering. You are *allowed* to spend this budget to move fast and release features, acknowledging that 100% reliability is impossible and overly expensive.
* **The Policy:** If the error budget is depleted for a given rolling window (e.g., 28 days), feature releases are halted. 100% of engineering effort shifts to reliability, testing, or tech debt until the budget recovers.

## Principal Engineer Interview Context
* **Defining SLIs:** Distinguish between Request-Driven systems (focus on availability, latency) and Data-Processing pipelines (focus on freshness, correctness, throughput).
* **The "Nines" Trap:** Do not aim for 100% availability. It is astronomically expensive, heavily restricts deployment speed, and users' ISPs/mobile networks usually have lower availability than your service anyway. 99.9% (3 nines) is ~43 minutes of downtime per month.
