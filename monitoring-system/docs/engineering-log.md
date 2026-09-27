# Failure-Driven Engineering Log

Every week, answer all six questions below and append a new dated entry.
Do not delete previous weeks' entries.

---

## Week 1 — Distributed OS Foundation

**Date:** <fill in>

**What changed?**
Built the first prototype: two edge nodes (edge-node-a, edge-node-b) and
one core node, each running an agent that reads CPU/memory and reports
to a central collector over HTTP every 5 seconds. Collector exposes
`/report`, `/status`, and `/health` endpoints.

**What failed?**
(Fill in once you've run it — e.g. "All three nodes reported identical
memory percentages because psutil read host-wide memory instead of the
container's own usage.")

**Why did it fail?**
(e.g. "psutil.virtual_memory() reports the whole machine, not the
cgroup the container is confined to.")

**How was it fixed?**
(e.g. "Read /sys/fs/cgroup/memory.current and memory.max directly, and
set explicit mem_limit/cpus per container in docker-compose.yml so
each node has a genuinely different resource ceiling.")

**What alternative was considered?**
(e.g. "Considered using docker stats via the Docker API from inside
the collector instead of self-reporting agents. Rejected for Week 1
because it centralizes measurement logic instead of distributing it,
which defeats the purpose of the exercise.")

**What was learned?**
(e.g. "Containerized 'per-node' metrics require explicit resource
limits and cgroup-aware reading; naive psutil calls silently return
host-level data.")

---

## Week 2 — Distributed Processing and Performance

**Date:** <fill in>

**What changed?**
Agents no longer send one raw reading per report. Each node now
collects a local WINDOW_SIZE of CPU/memory samples and computes its
own min/avg/max before sending a single aggregated summary to the
collector — this is the distributed-processing element for this
milestone: computation (aggregation) happens on the node, not
centrally. Added `experiments/run_experiment.py`, which polls
`/status` over a fixed duration and derives throughput, per-node
latency, and jitter, and `experiments/plot_results.py` to graph them.

**What failed?**
(Fill in once you run it — e.g. "Latency looked artificially large at
first because agent 'sent_at' used time.time() (local clock, no
timezone) while the collector's 'received_at' used an ISO UTC
timestamp; comparing them directly gave wrong latency values.")

**Why did it fail?**
(e.g. "Clock representations didn't match: Unix timestamp vs ISO 8601
string in different timezones/formats.")

**How was it fixed?**
(e.g. "Standardized on: agent sends `sent_at` as a raw Unix timestamp;
collector's `received_at` is parsed back into a Unix timestamp with
`datetime.fromisoformat(...).timestamp()` before subtracting, in
`run_experiment.py`.")

**What alternative was considered?**
(e.g. "Considered using NTP-synced clocks or a monotonic clock shared
via the network for stricter latency accuracy. Rejected for Week 2 as
overkill on a single Docker host where all containers share the same
system clock; revisit if nodes ever run on physically separate
machines.")

**What was learned?**
(e.g. "In distributed systems, timestamp format and clock
synchronization matter as much as the logic itself — mismatched
timestamp formats silently produce plausible-looking but wrong
latency numbers.")

---

<!-- Copy the block above for Weeks 3–12 -->
