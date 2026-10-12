# Portfolio Growth System: How to Add Case Study #2 & Beyond

> *"A portfolio that never gets a second project goes stale and stops proving anything new. The difference between a class artifact and a career platform is one simple habit, set up now while you still remember how everything works."*

---

## 1. How to Add the Next Case Study: The 3-Beat Playbook

### Exact File Destinations
- **Live HTML Page:** `docs/cases/case-02.html` (served via GitHub Pages).
- **Index Listing:** Link card added to `docs/index.html` under a `#case-studies` navigation section.
- **Figures & Visuals:** `docs/figures/case_02_*.png` and `work/figures/case_02_*.png`.
- **Reproducible Notebook:** `work/notebooks/case_02_rover_telemetry.ipynb`.
- **Metrics Receipts:** Committed JSON file at `work/outputs/case_02_metadata.json`.

---

### The Week 2 Three-Beat Story Shape
Every case study follows the proven three-beat structure:

```
[Beat 1: The Problem]
  ↳ Concrete operational scenario (not an abstract idea).
  ↳ The existing rule / heuristic baseline and why it fails in production.
  ↳ The asymmetric business cost of false alarms vs. false negatives.

[Beat 2: What You Did]
  ↳ Data provenance, sampling window, volume, and deliberate privacy exclusions.
  ↳ Feature engineering: strictly pre-decision telemetry (zero future-leakage).
  ↳ Honest validation design: grouped by entity / run / client (not naive random splits).
  ↳ The model architecture tested against the production rule baseline.

[Beat 3: What Came of It]
  ↳ Honest metrics table vs. baseline on the same held-out split (with base rate visible).
  ↳ Key visual figure with one bold takeaway sentence underneath.
  ↳ Permutation feature importances: what the model actually learned.
  ↳ Action Playbook: 4–5 discrete human-in-the-loop archetypes with reason codes.
  ↳ What we refuse to automate + boundary conditions + honest claim language.
```

---

### The 30-Minute Technical Routine to Ship
1. **Develop in Notebook:** Duplicate `work/notebooks/capstone.ipynb` $\to$ `work/notebooks/case_02_rover_telemetry.ipynb`. Run top-to-bottom so all cell outputs are rendered.
2. **Export Visuals & Receipts:** Save high-res PNG plots into `docs/figures/` and metrics into `work/outputs/`.
3. **Generate Static Page:** Duplicate `docs/index.html` structure $\to$ `docs/cases/case-02.html`. Update text, tables, and image links.
4. **Leakage & CI Safety Check:** Run `git status` and verify zero raw datasets are tracked:
   ```bash
   python -c "import glob, json; print([p for p in glob.glob('work/notebooks/*.ipynb') if not any(c.get('outputs') for c in json.load(open(p, encoding='utf-8'))['cells'] if c.get('cell_type')=='code')])"
   ```
5. **Deploy & Announce:** Commit and push to `origin main`. GitHub Pages automatically updates in ~60 seconds.

---

## 2. The Named Next Piece of Real Work

- **Project Title:** *Real-Time Telemetry Drift & Anomaly Triage for Autonomous Embedded Systems (Lab Rover)*
- **Domain:** Embedded Systems & Edge Machine Learning
- **Related Workspace:** `g:/Course's/Embedded System Lab/Project/rover`
- **Data Source:** High-frequency multi-sensor telemetry log (50 Hz) from autonomous rover obstacle-traversal runs:
  - 9 sensor channels: IMU acceleration ($a_x, a_y, a_z$), angular velocity ($\omega_z$), wheel encoder ticks, motor current draw ($I_{left}, I_{right}$), ultrasonic distance ($d$), battery voltage ($V_{bus}$).
- **The Operational Problem (Beat 1):**
  - Current rover firmware uses static threshold triggers (e.g. `motor_current > 3.0A` or `pitch > 25°`) to halt motors.
  - On rough terrain, transient current spikes trigger frequent **false alarms** (stopping the vehicle unnecessarily), while subtle multi-sensor drift (slipping wheels under normal current) fails to trigger alarms until the rover is mechanically stuck.
- **What We Will Do (Beat 2):**
  - Engineer 9 pre-decision rolling window features (jerk, power draw delta, slip ratio, vibration entropy).
  - Train an interpretable **HistGradientBoostingClassifier / Isolation Forest** to predict impending motor stall or mechanical slip 1.5 seconds in advance.
  - Enforce **Grouped Run Validation** (`GroupShuffleSplit` by `terrain_run_id`) to ensure models generalize to completely unseen surface topographies.
- **What Comes of It (Beat 3):**
  - Benchmark vs. static threshold rule on Precision@K and False Alarm Rate.
  - Produce a 4-Tier Rover Failsafe Playbook:
    1. *Active Traction Redistribution* (P1 - High slip)
    2. *Torque Throttling* (P2 - Transient stall risk)
    3. *Autonomous Emergency Halt* (P3 - Multi-sensor collision divergence)
    4. *Normal Terrain Cruise* (P4 - Monitor only)
  - Deploy quantized TFLite / Micro-C inference engine directly on microcontrollers.

---

## 3. Evidence of Concrete Reminder Set

- **Target Ship Date:** **Saturday, November 14, 2026 at 10:00 AM UTC+6**
- **Calendar Event Title:** `Ship Case Study #2: Autonomous Rover Telemetry Drift & Anomaly Triage`
- **Calendar Artifact:** Concrete iCalendar file created and committed at [`work/next_case_reminder.ics`](file:///E:/FlyRank/flyrank-ml-internship-starter/work/next_case_reminder.ics).
- **Recurrence:** Monthly recurring review nudge (every 4 weeks) to inspect ongoing portfolio projects.
- **Notification Setting:** 30-minute pop-up alarm + email trigger.

---

## 4. Preserved AI Build Context (Claude / Antigravity Project Kit)

Keep this project context preserved in your AI workspace so generating the next case study takes a 15-minute conversation rather than a full rebuild:

### Identity & Tone Kit
- **Author Identity:** KMSiam — Machine Learning Engineer & Embedded Systems Developer.
- **Portfolio Domain:** [https://kmsiam.github.io/Flyrank-Internship-ML/](https://kmsiam.github.io/Flyrank-Internship-ML/)
- **Repository:** [https://github.com/KMSiam/Flyrank-Internship-ML](https://github.com/KMSiam/Flyrank-Internship-ML)
- **Claim Discipline:** Strictly use *observed*, *measured*, *directional*, *decision-support*. Never use ungrounded causal words (*proves*, *causes*, *guarantees*). Report task base rate alongside all Precision@K and ROC-AUC numbers.
- **Design Tokens:** Deep slate `#0f172a`, Royal blue `#1d4ed8`, Emerald `#059669`, system fonts, high-contrast accessible cards, embedded figures with bold captions.

### Quick-Start Prompt for Next Case Study
> *"Hey Antigravity/Claude — let's build Case Study #2 using our established portfolio platform rules in `work/NEXT_CASE_STUDY.md`. The project is Autonomous Rover Telemetry Drift Triage. Here is the dataset schema: [paste columns]. Follow our Week 2 three-beat shape (Problem, What we did, What came of it), run GroupShuffleSplit by run_id, benchmark against the heuristic current-threshold rule, and output the HTML card and notebook."*
