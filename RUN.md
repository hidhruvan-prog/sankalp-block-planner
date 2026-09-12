# Run SANKALP on your own laptop

Two ways. The first needs nothing installed at all.

---

## 1. Just look at it (no install, 5 seconds)

Unzip `SANKALP-source.zip` and double-click **`SANKALP/OFFLINE_DEMO.html`**.

That is the whole console — both plans, the Gantt charts, the model scorecard — rendered
from plans the solver already produced. No server, no Python, no internet. The buttons
under **"Try the alternatives"** switch between four *real* solver runs (drop the AI
curve, blind it to routes, make due dates a soft penalty) and redraw every number.

Same page online: <https://hidhruvan-prog.github.io/sankalp-block-planner/>

---

## 2. Actually run the optimiser

### What you need

**Python 3.12, 3.13 or 3.14** — check with `python --version` (`python3 --version` on
macOS/Linux). 3.11 is too old (numpy 2.5.1 needs 3.12+) and 3.15 does not work yet
(OR-Tools has no 3.15 wheel). No GPU needed; everything runs on a laptop CPU.

Below, Windows users type `python`, macOS/Linux users type `python3`.

### Install

Unzip the file, open a terminal **inside the `SANKALP` folder**, then:

```bash
python -m venv .venv
.venv\Scripts\activate                 # Windows
# source .venv/bin/activate            # macOS / Linux

python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
python -m pip install -r requirements.txt
```

The virtual environment matters: `requirements.txt` pins 50-odd exact versions, and without
it they overwrite whatever your other Python projects use. (On recent Debian/Ubuntu and
Homebrew Python, pip *refuses* to install without one.)

Install torch **first**, from the CPU index. Do it the other way round and pip pulls the
CUDA build — a ~2.5 GB download you have no use for.

### Start the console

```bash
python -m uvicorn api:app --port 8000
```

Open <http://127.0.0.1:8000/>. It loads instantly from the cached plan.

If you see `error while attempting to bind on address ... only one usage of each socket
address`, port 8000 is already taken — use `--port 8001` and open that instead.

Press **Re-solve live**: it retrains the model and re-solves from scratch, about 90
seconds. The solve is deterministic, so it lands on exactly the same plan and the page
says so. Same inputs, same plan, every time — that is the audit trail.

Windows shortcut: double-click **`START_DEMO.bat`** (it starts the server and opens the
browser; leave the black window open).

### Prove it works

```bash
python run_demo.py
```

Re-solves from scratch on the command line, then tells you whether it reproduced the
canonical plan exactly. It should end with `IDENTICAL: same inputs, same plan` and
`objective 8387.0`. Takes about 70 seconds.

```bash
python selfcheck.py
```

Runs both planning horizons end to end, validates every plan with a checker that imports
nothing from the solver, and compares against a frozen golden file. About 8 minutes. Ends
with `ALL CHECKS PASSED`, plus one `NOTE` we deliberately do not hide (28-day train paths
come out slightly worse than the baseline).

```bash
python -m sankalp.benchmarks   # ~1 min: LightGBM vs Cox vs SANKALP-Net, one temporal split
python build_scenarios.py      # ~6 min: re-solve the four scenarios in the page
python experiments.py          # ~30 min: the seeded A/B runs behind every claim we make
```

*Exact reproduction caveat: the plan depends on the trained network, so it is identical for
a given PyTorch build. A different torch version can shift the weights slightly and move
the objective. `requirements.txt` pins the version this was verified with.*

---

## What you are looking at

A **block** (line possession) is a window where a stretch of track is handed to a
maintenance department and no trains run on it. Engineering, S&T and Traction Distribution
each ask for them separately today, so the same section gets shut more than once a week.

SANKALP plans all three departments as one problem:

1. **SANKALP-Net** (PyTorch: graph-convolution encoder + one survival head per department)
   gives every defect a day-by-day failure probability over the next 30 days.
2. **Stage 1** of the CP-SAT solver proves the fewest due-date days late the crews and
   traffic windows physically allow — using no AI at all.
3. **Stage 2** minimises failure risk by start day, trains cancelled or diverted, and paths
   lost, without ever leaving that floor.
4. An **independent validator** re-checks the finished plan, due dates included, and its
   count must match the solver's own.

**Everything is simulated** — the division, timetable, crew limits, defects and due dates
come from a documented generator, not from Indian Railways. The "rule-based baseline" it is
measured against is our own model of current practice. Only the relative comparison, scored
identically, is claimed.

Team CodeCraft · Smart India Hackathon 2026 · Problem Statement 26027 · Ministry of Railways
