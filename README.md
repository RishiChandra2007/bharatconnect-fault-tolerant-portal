## 📐 Retry Math & Algorithmic Design
To prevent client synchronization during gateway timeouts, client retries utilize **Exponential Backoff with Full Jitter**:

$$Delay = \text{random}(0, \text{Base Delay} \times 2^{\text{attempt}})$$

- **Attempt 1:** $\text{random}(0, 1000\text{ms})$
- **Attempt 2:** $\text{random}(0, 2000\text{ms})$
- **Attempt 3:** $\text{random}(0, 4000\text{ms})$

By picking a uniform random duration between 0 and the exponential cap, retry requests are staggered evenly across time rather than slamming the server in synchronized waves.

## 📊 Empirical Load Testing
The simulator was benchmarked using a bulk load script simulating **500 concurrent virtual clients** firing **1,000+ requests** against the active 60% fault-injection layer:

| Metric Observed | Unthrottled Retries | Exponential Backoff + Jitter |
| :--- | :--- | :--- |
| **Retry Strategy** | Immediate / Fixed Interval | Exponential Backoff + Full Jitter |
| **Traffic Profile** | High concurrency spikes | Staggered, smooth retry distribution |
| **System Impact** | CPU thrashing, immediate socket exhaustion | Controlled request flow, stabilized gateway pressure |

### 🎯 Key Engineering Takeaways
- **Traffic Shaping vs. Gateway Throughput:** Under extreme server saturation, overall request throughput is bounded by hardware capacity (~400 req/sec). While backoff cannot invent extra server capacity, it eliminates thundering-herd spikes and spreads incoming load predictably over time.
- **Resilience Validation:** Demonstrates how client-side backoff paired with randomized jitter protects upstream infrastructure during legacy backend outages.

## 🔌 API Endpoints
- `POST /api/recharge` – Simulates a payment transaction; subject to the 60% 504 failure rate.
- `GET /api/metrics` – Fetches live gateway telemetry (successes, drops, retry counts) for the UI.
- `POST /api/reset` – Flushes system metrics and resets simulation counters.

## 💻 Tech Stack
- **Frontend:** HTML5, Tailwind CSS, Live Metrics Polling
- **Backend:** Node.js, Express.js (v5)

## 🚀 How to Run Locally
1. Clone the repository and install dependencies:
   ```bash
   npm install