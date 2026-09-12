# Run SANKALP on your own laptop

Two ways. The first needs nothing installed.

---

## 1. Just look at it (no install, 5 seconds)

Double-click **`OFFLINE_DEMO.html`**. That is the whole console — the plan, both Gantt
charts, the model scorecard — rendered from a plan the solver already produced. No server,
no Python, no internet.

Online version of the same page: <https://hidhruvan-prog.github.io/sankalp-block-planner/>

---

## 2. Actually run the optimiser (about 10 minutes of setup)

You need **Python 3.11 or newer** (`python --version`). Then, in this folder:

```bash
python -m pip install -r requirements.txt
python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
```

The second line gets the CPU build of PyTorch (~200 MB) instead of the CUDA one (~2.5 GB).
You do not need a GPU — everything here runs on a laptop CPU.

### Start the console

```bash
python -m uvicorn api:app --port 8000
```

Open <http://127.0.0.1:8000/>. It loads instantly from the cached plan.

Press **Re-solve live** and it trains the model and re-solves from scratch — about two
minutes. The solve is deterministic, so it lands on exactly the same plan, and the page
tells you so. That is the point: same inputs, same plan, every time.

On Windows you can double-click **`START_DEMO.bat`** instead, which does both steps.

### Prove it works

```bash
python selfcheck.py
```

Runs both planning horizons end to end, validates every plan with a checker that imports
nothing from the solver, and compares the baseline against a frozen golden file. Takes
about 8 minutes. Prints `ALL CHECKS PASSED`, plus one `NOTE` we deliberately do not hide
(28-day train paths).

```bash
python run_demo.py
```

Re-solves from scratch on the command line and tells you whether it reproduced the
canonical plan exactly (it should: `objective 8387`). About 3 minutes.

```bash
python experiments.py        # ~30 min: the controlled A/B runs behind the deck's claims
python -m sankalp.benchmarks # ~1 min: LightGBM vs Cox vs SANKALP-Net on one temporal split
```

---

## What you are looking at

A **block** (line possession) is a window where a stretch of track is handed to a
maintenance department and no trains run on it. Engineering, S&T and Traction Distribution
each ask for them separately today, so the same section gets shut more than once a week.

SANKALP plans all three departments as one problem:

1. **SANKALP-Net** (PyTorch: graph-convolution encoder + one survival head per department)
   gives every defect a day-by-day failure probability over the next 30 days.
2. **Stage 1** of the CP-SAT solver proves the fewest due-date days late the crews and
   traffic windows physically allow, using no AI at all.
3. **Stage 2** minimises failure risk by start day, trains cancelled or diverted, and paths
   lost — without ever leaving that floor.
4. An **independent validator** re-checks the finished plan, due dates included.

**Everything is simulated** — the division, timetable, crew limits, defects and due dates
come from a documented generator, not from Indian Railways. The "rule-based baseline" it is
measured against is our own model of current practice. Only the relative comparison,
scored identically, is claimed.

Team CodeCraft · Smart India Hackathon 2026 · Problem Statement 26027 · Ministry of Railways
