# Distributed Network Monitoring System — Week 1 Baseline

ICS 2403 — Theme 2: Distributed Network Monitoring System

## What this is

A minimal distributed monitoring prototype:

- **edge-node-a**, **edge-node-b**, **core-node** — each runs an
  *agent* that reads its own CPU and memory usage and reports it to
  a central collector every 5 seconds.
- **collector** — receives reports over HTTP, stores the latest
  reading per node, and exposes `/status` for inspection.

```
edge-node-a ─┐
edge-node-b ─┼─> collector (HTTP :5000) ─> /status, /report, /health
core-node   ─┘
```

## Prerequisites

- Docker Desktop (with WSL2 backend) installed and running
- (Python is only needed on your host if you want to run scripts
  outside Docker; the containers bring their own Python 3.13)

## Running it

From the `monitoring-system` folder:

```powershell
docker compose up --build
```

This builds the collector and agent images and starts all four
containers. You'll see interleaved logs like:

```
collector-1    | [collector] received from edge-node-a (role=edge): cpu=1.1% mem=3.2%
edge-node-a-1  | [agent:edge-node-a] sent report (cpu=1.1% mem=3.2%) -> collector responded 200
```

## Checking status

In a separate PowerShell window:

```powershell
Invoke-RestMethod http://localhost:5000/status | ConvertTo-Json -Depth 5
```

You should see `known_nodes`, a `latest` reading per node, and
`total_records_received`.

## Stopping it

```powershell
docker compose down
```

## Project layout

```
monitoring-system/
  docker-compose.yml
  collector/
    app.py
    Dockerfile
    requirements.txt
  agent/
    app.py
    Dockerfile
    requirements.txt
  experiments/        <- put your Week 2+ measurement scripts here
  docs/
    engineering-log.md
    reproducibility.md
```

## Why memory/CPU differ per node now

Each container has an explicit resource limit set in
`docker-compose.yml` (`mem_limit`, `cpus`), and the agent reads
`/sys/fs/cgroup/memory.current` and `memory.max` to report **this
container's own** usage rather than the host machine's. This fixes
the earlier issue where every node reported identical memory.

## Week 2 — Distributed Processing and Performance

Each agent now aggregates a local window of samples (default 5)
before sending a summary to the collector — this is the "distribute
computation" requirement. The collector also timestamps when it
receives each report so latency can be measured.

### 1. Start the system (if not already running)
```powershell
docker compose up --build
```

### 2. Set up a Python environment on your HOST for the experiment scripts
(separate from the containers — these run on your laptop, not in Docker)
```powershell
cd monitoring-system
python -m venv venv
venv\Scripts\activate
pip install -r experiments/requirements.txt
```

### 3. Run an experiment while the system is running
In a new terminal (with the venv activated):
```powershell
python experiments/run_experiment.py --duration 60 --label window5
```
This polls `http://localhost:5000/status` once per second for 60
seconds and writes `experiments/results/window5.csv`, plus prints a
summary of latency, jitter, and throughput per node.

### 4. Try a different load level
Edit `WINDOW_SIZE` in `docker-compose.yml` for each node (try `2` for
faster reporting / higher load, or `10` for slower / lower load), then:
```powershell
docker compose up --build -d
python experiments/run_experiment.py --duration 60 --label window2
```
Repeat for as many settings as you want to compare.

### 5. Plot the results
```powershell
python experiments/plot_results.py --files window5 window2
```
This produces `throughput.png` and one `latency_<node>.png` per node
in `experiments/results/`, ready to paste into your report.

## Next steps (Week 3+)

See `docs/engineering-log.md` for the weekly log template and
`docs/reproducibility.md` for the environment record to keep updated.
