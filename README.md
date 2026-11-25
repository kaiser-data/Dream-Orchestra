<div align="center">

# 🤖 Dream Orchestra

### Dream Orchestra turns Dream-1 skills into machine workflows and measures whether they actually work.

**Draw the process as a workflow: robot skills, machine steps, sensors, error paths.**<br>
**Run it in a simulated cell, and fine-tune Dream-1 only when the numbers say it pays off.**

**Measured:** a cell fine-tune lifts success from **6/40 to 14–17/40** (same recipe, three trainings), but forgets
a skill outside the cells (**6/20 → 0/20**) and does not transfer to a new task (**0/20**).<br>
So the app decides per task which checkpoint may run.

[![Live app](https://img.shields.io/badge/live%20app-dream--orchestra.vercel.app-111?style=for-the-badge&logo=vercel)](https://dream-orchestra.vercel.app)
[![Results](https://img.shields.io/badge/results-live%20page-2a78d6?style=for-the-badge)](https://cell-finetune-results.martinkaiser-bln.workers.dev)
[![Demo video](https://img.shields.io/badge/demo%20video-2%3A42-e8590c?style=for-the-badge&logo=youtube&logoColor=white)](https://cell-finetune-results.martinkaiser-bln.workers.dev/dream-orchestra-demo.mp4)

![Dream-1](https://img.shields.io/badge/Dream--1-skills%20·%20fine--tuning-1c5cab)
![MuJoCo](https://img.shields.io/badge/MuJoCo-two%20simulated%20cells-555)
![FastAPI](https://img.shields.io/badge/FastAPI-workflow%20engine%20·%20simulation%20API-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-React%20Flow%20editor-61dafb?logo=react&logoColor=black)
![Tests](https://img.shields.io/badge/tests-93%20backend%20·%2086%20frontend-2ea44f)

🥉 **3rd place, Dream Labs track** · REMix × Dream Labs hackathon ([Robotics × AI HackLab #2](https://luma.com/h0mcaqio)), CIC Berlin, 21–22 Nov 2026 · [Results page](https://cell-finetune-results.martinkaiser-bln.workers.dev) ·
[Demo video (2:42)](https://cell-finetune-results.martinkaiser-bln.workers.dev/dream-orchestra-demo.mp4)

</div>

> **Try it online (no install):** [dream-orchestra.vercel.app](https://dream-orchestra.vercel.app) (frontend on
> Vercel, backend API on Railway in `mock` mode). Load an example in the workflow editor, press **Start run**, then
> open **Robot & Fine-tune** for the measured results and the decision table. The hosted backend has no MuJoCo cell,
> so runs are replayed by a stand-in and the banner says **SIMULATED API**; the live cell needs the local setup below.
>
> **Try it (local, about 5 minutes, no Dream credits):** `DREAM_API_MODE=sim docker compose up -d --build`, open
> `http://localhost:8080`. In the workflow editor pick **Load example → CNC-Zyklus (Zelle)** and press
> **Start run**; the live image of the cell appears next to the log. Then load **CNC-Zyklus: Fehlerfall**: the unload
> step fails, the workflow takes its error path and switches the stack light to red. In `sim` mode the arm is moved
> by the scripted reference controller, not by Dream-1, and the banner says so: model results in the app are
> precomputed measurements of the trained checkpoints.

## Findings in one minute

| # | Finding | Evidence (simulation, same test scenes, paired) |
|---|---|---|
| 1 | **Fine-tuning on cell demos works, and it repeats.** | Base 6/40 → same recipe trained three times: 14, 17, 16 of 40 (p = 0.057, 0.013, 0.021) |
| 2 | **More training is not better.** | 1000 steps: 13/40, no better than 500 steps at twice the cost; mixing in 50 % base data: 6/40 |
| 3 | **The fine-tune forgets other skills.** | Picking a banana outside the cells: 6/20 → 0/20 (6 : 0, p = 0.031), pre-registered "harms" rule met |
| 4 | **It does not transfer to a new task.** | Untrained `cnc_return`: expert 20/20, base 1/20, fine-tune 0/20 → "transfer not shown" |
| 5 | **The scripted controller still wins in the cell.** | Expert 39/40; the best fine-tune solves less than half |

**Conclusion:** a fine-tuned checkpoint is a tool for a specific robot and cell, not an upgrade. It belongs in the
tasks where it was measured better, and nowhere else. Dream Orchestra makes that decision visible per task, from
measurements, before a checkpoint runs in a workflow. Limits: simulation only, 10 scenes per task, contacts rise
with fine-tuning, so no safety claim.

![How it works: build the workflow in the editor, the engine runs it against a MuJoCo cell; a scripted expert records demos, Dream-1 is fine-tuned, every checkpoint is measured on 40 identical scenes, and a decision table says which checkpoint may run which task](docs/img/how-it-works.svg)

![Workflow editor with the CNC cycle: load blank, start machine, wait for "machine done", unload, green light, error paths to a red stack light; below, the run log, the live image of the CNC cell and the cell's devices](docs/img/editor-live-cell.png)

## The idea

A Dream-1 skill is a prompt that moves a robot. A factory needs more than that: the robot step has to wait for a
machine, react to sensors, and do something sensible when it fails. And before a learned skill runs in a real cell,
someone has to know how often it succeeds there. Today that means custom glue code and gut feeling.

| Without | With Dream Orchestra |
|---|---|
| Robot prompt and machine logic in separate scripts | One workflow: skills, machine steps, sensor waits, conditions, loops, error paths |
| A failed grasp stops everything, silently | Every step has a time limit, retries and an error path; the log shows API status, duration and credits |
| "The fine-tune looks better in the video" | Success, contacts and time measured on the same test scenes, paired, with the noise of repeat runs |
| Fine-tune first, ask later | Measure the base in your cell first; train only where it is weak |

## How it works

1. **Skills**: a Dream-1 prompt per robot type, e.g. *"put the metal block on the machine tray"*.
2. **Workflow**: the visual editor chains skills with *Aktor setzen*, *Warte auf Sensor*, *Bedingung* and an error
   path. The backend engine runs it (pause, single step, STOP at any time; STOP during a cell step also halts the
   cell). Run it twice in a row: the second load restarts the cell with a fresh blank (the simulation has no feeder).
3. **Cell and devices**: sensors and actuators are devices with an adapter (virtual today, MQTT and OPC UA settings
   prepared). In `sim` mode the cell devices (`zelle-*`) mirror a live MuJoCo cell: CNC machine with door, tray and
   stack light, or a sorting conveyor.
4. **Measured fine-tune**: the scripted expert records checked demonstrations, a gated script uploads them and starts
   the Dream-1 fine-tune only if replay and validation pass, and the evaluation compares base and fine-tune on 40
   identical scenes.

| Execution | Who moves the arm | Labelled in the log as |
|---|---|---|
| `mock` backend | nobody (stand-in) | banner "SIMULATED API" |
| `sim` backend, cell task | scripted reference controller | **Referenz-Controller** |
| `real` backend, cell task | only with checkpoint `referenz` (explicit); a learned checkpoint stops the block with a clear message | **Referenz-Controller** |
| `real` backend, other skills | Dream-1 in MuJoCo (costs credits) | **Simulationslauf** |

**Failure handling in the cell:** an actuator block waits for the cell's acknowledgement (a blocked cycle start, e.g.
no blank on the tray, takes the error path); cell sensors report *not available* instead of an old value when the
cell is unreachable or the last reading is older than 2 s. Dream-1 itself runs in the cells in the evaluation
(`src/Simulation/tools/eval_cells.py`), not yet inside a workflow.

## Results (simulation, first attempt per scene)

From [`docs/results/report_corrected.md`](docs/results/report_corrected.md), 40 identical test scenes per model,
4 tasks × 10 seeds, never used in training.

| Model | Success | Paired vs base | Solved within 30 s |
|---|---|---|---|
| Scripted expert (reference) | 39/40 | – | 39/40 |
| Dream-1 beta, two runs | 6/40, 7/40 | 11 scenes flip between the runs | 4, 2 |
| Fine-tune, 160 cell demos | 14/40 | 11 : 3, p = 0.057 | 13 |
| Same recipe, second training | 17/40 | 14 : 3, p = 0.013 | 12 |
| Same recipe, third training | 16/40 | 13 : 3, p = 0.021 | – |
| Learning rate 3e-5 | 20/40 | 19 : 5, p = 0.007 | 20 |
| 50 % Dream base data mixed in | 6/40 | 6 : 6, p = 1.0 | 5 |
| Twice the steps (1000) | 13/40 | 10 : 3, p = 0.092 | – |

The same recipe trained three times gives 14, 17 and 16 of 40, against 6 for the base: the gain is repeatable.
Twice the training steps did not help. The app's *Robot & Fine-tune* tab turns these numbers into a decision:
which checkpoint may run which task (`GET /api/results/policy`).

**Limits:** simulation only; runs with arm–cell contact go up with fine-tuning (8 → 13–17 of 40), so no safety
claim; per-task results are not stable across trainings. The cell fine-tune forgets outside the cells: banana
6/20 (base) → 0/20, 6 : 0 paired, which meets the pre-registered "harms" rule, so cell checkpoints stay cell-only. **Transfer test** (pre-registered in [`docs/results/TRANSFER_PREREG.md`](docs/results/TRANSFER_PREREG.md)): on
`cnc_return`, a similar CNC task never trained, the expert solves 20/20, the base 1/20 and the c500 fine-tune 0/20
(paired 0 : 1). By the pre-registered rule: **transfer not shown**. The fine-tune helps on the trained tasks only.

![Decision table in the app: per task, who runs it today, the base model's score and the approved fine-tune; the banana row shows the cell fine-tune dropping 6/20 to 0/20, so it is excluded there; below, every other fine-tune with its 40-scene score and why it was not approved](docs/img/decision-table.png)

**Check the numbers yourself:** every scored scene is committed in [`docs/results/raw/`](docs/results/raw/)
(one JSON line per scene and attempt). `CELLS_DIR=docs/results/raw python3 src/Simulation/tools/results_page.py
/tmp/results.html` recomputes the table above (first attempt per scene, paired McNemar) without Docker or network.
The app's results tab reads the same snapshot ([`docs/results/results.json`](docs/results/results.json)) when no
local measurements exist.

## Questions we expect

- **"Your scripted expert gets 39/40. Why Dream-1?"** The expert is programmed per cell and task; it is how we
  produce demonstrations and a reference. Dream-1 learns from demonstrations. The platform's job is to show, per
  robot type and cell, whether a learned skill is good enough, before anyone relies on it.
- **"Is it safer after fine-tuning?"** No evidence for that; contacts go up. We report it.
- **"Does it work on a real robot?"** Not tested. Everything here is MuJoCo.

![Cell control room: live view of the CNC cell with tasks, controls and sensor states](docs/img/cell-control-room.jpg)

---

## What's next

Dream-1 checkpoints executing cell tasks inside a workflow (today: scripted controller) · real MQTT / OPC UA
devices · a part feeder in the simulated cell · more test scenes per task so per-task decisions become possible

---

Built in two days at the REMix × Dream Labs hackathon, Berlin. Tools: Dream-1 (fine-tuning API), MuJoCo,
FastAPI, React Flow, Claude Code. Video narration: ElevenLabs and Qwen3-TTS (AI voices); footage is real app and
MuJoCo recordings. Never commit API keys: they live in `.env`, which stays out of git.

## Details (Deutsch)

Docker Compose startet das Frontend mit Nginx, das Backend mit Python 3.12, den Simulationsdienst mit Python 3.12, MuJoCo und FastAPI und den SQLite-Datenbankdienst. Die API des Simulationsdienstes beschreibt [`src/Simulation/README.md`](src/Simulation/README.md), die Live-Doku liegt unter `http://localhost:3001/docs`.

### Setup

Docker starten und in den Repo-Ordner wechseln (dort liegt `compose.yaml`). Alle Dienste starten mit `docker compose up`. Das Backend startet ohne Angabe im Modus `mock` (ohne Simulation); für die Zellen-Demo `DREAM_API_MODE=sim docker compose up -d --build` verwenden, `real` kostet Dream-Credits. Mit `--build` werden die Images vorher gebaut.

```sh
docker compose up --build
```

Für einen einzelnen Dienst beispielsweise `docker compose up --build backend` ausführen. Die Dienstnamen und lokalen Adressen stehen in der Tabelle.

| Dienst | Quellordner mit Dockerfile | Adresse |
| --- | --- | --- |
| `frontend` | `src/Frontend` | `http://localhost:8080` |
| `backend` | `src/Backend` | `http://localhost:8000/health` |
| `simulation` | `src/Simulation` | `http://localhost:3001/health`, API-Doku `http://localhost:3001/docs` |
| `simulation-gui` (optional, `--profile gui`) | `src/Simulation` | `http://localhost:6080/vnc.html` (MuJoCo-Fenster im Browser) |
| `database` | `src/Database` | `http://database:8081/health` im Compose-Netz |

Compose bindet jeden Quellordner schreibgeschützt ein. Nach Änderungen am Backend oder Simulationsserver den Dienst neu starten, beispielsweise mit `docker compose restart backend`. `docker compose down` stoppt die Dienste.

### SQLite und Datenmodell

Der Dienst `database` legt beim ersten Start `src/Backend/data/dreamorchestra.sqlite3` an. Die Datei bleibt nach `docker compose down` auf dem Host erhalten und wird von Git ignoriert. Über `http://localhost:8000/health/database` prüft das Backend die Verbindung zum Datenbankdienst.

Das Schema liegt in [`src/Backend/models/schema.sql`](src/Backend/models/schema.sql) und enthält als Ausgangspunkt die Tabelle `skills`. Der Datenbankdienst führt das Schema bei jedem Start aus. Neue Tabellen mit `CREATE TABLE IF NOT EXISTS` ergänzen und `docker compose restart database` ausführen. Änderungen an vorhandenen Tabellen brauchen eine gezielte Migration.

Für SQL-Abfragen die SQLite-Shell im Datenbankcontainer öffnen. [Python-Dokumentation zur SQLite-Shell](https://docs.python.org/3.12/library/sqlite3.html#command-line-interface)

```sh
docker compose exec database python -m sqlite3 /data/dreamorchestra.sqlite3
```

### Dream-Labs-Verbindungstest

Falls `.env` fehlt, `.env.template` als `.env` kopieren. Dann `DREAM_API_KEY` eintragen. Der Test ruft den Modellkatalog ab und zeigt SDK-Version und Anzahl der Checkpoints an. [Dream-Labs-Plattform](https://platform.dreamlabs.ai/)

```sh
docker compose run --rm --build backend python test_dream_api.py
```

In PowerShell startet `./start.ps1` den Verbindungstest auch ohne Compose.

### Umgebungsvariablen

Compose übergibt `.env` an den Backend-Dienst. Der Dream-Labs-Verbindungstest benötigt den API-Key. Werte als `NAME=Wert` ohne Anführungszeichen eintragen. Die optionalen URLs nur ändern, wenn Dream Labs für den Zugang andere Adressen vorgibt.

| Variable | Zweck | Standard |
| --- | --- | --- |
| `DREAM_API_KEY` | API-Key für die Authentifizierung | Pflichtfeld |
| `DREAM_API_URL` | Plattform mit dem Modellkatalog. Der Test verwendet diese URL. | `https://platform.dreamlabs.ai` |
| `DREAM_GATEWAY_URL` | Gateway für künftige Inferenzaufrufe | `https://api.dreamlabs.ai` |
| `DREAM_DATA_URL` | Dienst für Datensätze und Fine-Tuning-Jobs | `https://data.dreamlabs.ai` |

`.env` bleibt lokal. `.env.template` kann mit dem Projekt geteilt werden.
