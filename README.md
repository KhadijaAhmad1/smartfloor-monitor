# SmartFloor Monitor

**Live OEE, on-time delivery risk, and predictive analytics for manufacturing shop floors — built as a self-contained, offline-capable demonstrator.**


---

## The problem

Small and medium manufacturers running precision engineering and job-shop production often have no live visibility into their own production floor. A machine can sit idle for two hours before anyone notices. Priorities are set by instinct rather than data. Whether today's order book will hit its delivery commitments is something nobody finds out until it's already too late to act.

This is a well-studied problem in large automotive and aerospace plants with mature IT infrastructure. It is far less studied in the mixed-fleet SME context, a shop floor with machines spanning fifteen years of age, no existing data infrastructure, and a constrained budget for instrumentation. That gap is what this project explores.

## What this is

SmartFloor Monitor is a single-file, offline-capable web dashboard that demonstrates how a live machine signal can be turned into an actionable business decision, specifically, whether a time-sensitive order is at risk of missing its delivery commitment, and what to do about it right now.

It is built around one machine, a fibre laser cutting cell,  chosen because in a typical job-shop flow, the cutting stage is the pacemaker for everything downstream. If the laser falls behind, every subsequent process stage falls behind with it.

The dashboard has four views:

- **Live monitor** — real-time OEE (Availability × Performance × Quality), machine state, shift timeline, downtime Pareto, and live alerts
- **Historical data** — a 14-day shift summary with OEE and availability trends, and a cross-shift downtime Pareto
- **Predictions** — four explainable forecasting methods computed entirely client-side (see Methodology below)
- **Dataset** — the underlying CSV data, viewable and downloadable

## Why it matters

This isn't a static mockup. The on-time risk model, the OEE engine, and all four prediction methods are live, computed in the browser from embedded CSV data, and respond in real time to simulated machine events. You can inject a stoppage and watch a borderline order tip from "on track" to "at risk" to "late" in front of you, then watch the system generate a specific recommendation: which order to prioritise, and which lower-priority order to deliberately delay.

## Methodology

Every prediction method was deliberately chosen to be explainable rather than maximally sophisticated. In a real deployment, the people relying on this system, shop floor operators and supervisors, need to trust it and be able to reason about why it says what it says. A black-box model that nobody on the floor understands will not get adopted, no matter how accurate it is.

| Prediction | Method | Why this method |
|---|---|---|
| End-of-shift OEE forecast | Exponential smoothing (α = 0.45) blended with 14-day historical mean | Gives more weight to today's emerging pattern than to history, while staying interpretable in one sentence |
| On-time delivery confidence | Monte Carlo simulation, 2,000 runs sampling the historical availability distribution | Communicates uncertainty directly as a probability, rather than a single point estimate that hides how confident the system actually is |
| Predictive maintenance | Rolling mean of interval between maintenance events | No ML required, pure pattern recognition on data the machine already generates |
| Scenario planning | Historical mean downtime profile, recalculated under a hypothetical intervention | Direct arithmetic on real data; answers "what if we fixed X" without inventing model structure the data can't support |

A methodological note on validity: demonstrating an OEE improvement is genuinely caused by an intervention (rather than confounded by seasonal order mix, operator turnover, or the Hawthorne effect) requires a proper before/after design with tracked confounding variables, not just a before/after number. This project is structured as a foundation for that kind of evaluation, not a claim that it has already been done.

## Architecture

```
Machine signal  →  Edge device  →  Local store  →  Dashboard
(current-clamp     (Node-RED,       (SQLite /        (this repo —
 sensor or          MQTT broker)     time-series       vanilla JS,
 native controller)                  DB)               zero dependencies)
```

The capture layer is designed to work on legacy machines with no digital output, using a clip-on current sensor that requires no controller access and no machine downtime to install, as well as on newer machines with native Modbus or OPC-UA interfaces. Capture is always read-only.

## Dataset

Two CSV datasets are embedded in the dashboard and downloadable from the Dataset tab:

- `shifts.csv` — 15 rows, one per shift: date, operator, planned/runtime/downtime hours, sheets cut, parts cut, rejects, OEE factors, top downtime reason
- `events.csv` — 26 rows, one per machine event from a single shift: timestamp, event type, job reference, sector, material, cycle time, reason code

All data is synthetic, generated to reflect realistic values for a busy fibre laser cutting operation (availability ~80%, OEE 60–86%, reject rate ~1.9%). No production system was accessed to build this project; the structure reflects what a real deployment's capture layer would produce.

## Tech stack

Pure HTML, CSS and vanilla JavaScript. No build step, no framework, no external dependencies, the entire dashboard, including chart rendering, runs from a single file and works fully offline. This was a deliberate constraint: any environment that can open a web browser can run this, with no installation, no internet connection, and no compatibility risk.

## Running it

Clone the repo and open `smartfloor-monitor.html` in any modern browser. That's it.

```bash
git clone https://github.com/yourusername/smartfloor-monitor.git
cd smartfloor-monitor
open smartfloor-monitor.html
```

## Roadmap

This demonstrator represents the first phase of a larger applied research direction:

1. **Phase 1** — single-machine live monitoring and on-time risk prediction (this repo)
2. **Phase 2** — multi-machine deployment, cross-process flow visibility, predictive maintenance at fleet scale
3. **Phase 3** — anomaly detection on the live OEE stream; AI-assisted production scheduling once sufficient historical data has accumulated

## About

Built by Khadija Ahmad — AI Innovation Lead, with a background in computer science (BSc, First Class) and AI and data science (MSc, Distinction). This project sits alongside other applied AI and data engineering work exploring how manufacturing SMEs can build production-grade data infrastructure without the IT budgets of large enterprise plants.

## License

MIT
