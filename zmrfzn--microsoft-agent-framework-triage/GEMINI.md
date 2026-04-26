## microsoft-agent-framework-triage

> This project is a conference talk demo for **Experts Live India**.

# CLAUDE.md — MAF Incident Triage Talk (Experts Live India)

This project is a conference talk demo for **Experts Live India**.
Always read this file first before any code or slide work.

## Talk Details

- **Title:** Microsoft Agent Framework: The Open-Source Engine for Agentic AI Apps
- **Event:** Experts Live India
- **Format:** 30 minutes, Level 200
- **Demo style:** Pre-recorded demo + code tour (Streamlit app on `streamlit-demo` branch)
- **Language:** Python

## Project Structure

```
AgentFramework/
├── triage.py          ← pipeline entrypoint (ConcurrentBuilder + run_triage)
├── tools.py           ← search_logs · search_runbooks · score_severity
├── agents/
│   ├── log_agent.py        ← LogAnalysisAgent
│   ├── runbook_agent.py    ← RunbookRAGAgent
│   ├── severity_agent.py   ← SeverityScorerAgent
│   └── synthesis_agent.py  ← SynthesisAgent (incident commander)
├── app.py             ← Streamlit demo UI (streamlit-demo branch)
├── run_demo.sh        ← launch script (NR agent + SSL bypass)
├── newrelic.ini       ← NR agent config
├── data/
│   ├── alert.json     ← payment-api P1 incident (INC-20240315-0042)
│   ├── alert2.json    ← auth-service incident (INC-20240822-0117)
│   └── runbooks.json  ← 3 runbooks: OOMKilled, DB pool, circuit breaker
├── docs/
│   ├── antigravity/slidev/slides.md  ← Slidev presentation
│   └── foundry-tracing-investigation.md
└── .env.example
```

## Use Case: Multi-Agent Incident Triage Pipeline

Alert fires → three specialist agents run in **parallel** via `ConcurrentBuilder`
→ results converge into a synthesis agent → structured remediation report + OTel traces.

```
Alert
  │
  ├─[parallel]──────────────────────┐
  │                                 │
  ▼           ▼            ▼        │
LogAgent   RunbookAgent  SeverityAgent
  │           │            │
  └─[with_aggregator()]────┘
             │
             ▼
       SynthesisAgent
             │
             ▼
     Remediation Report
     + OTel trace spans
```

## MAF Patterns Used

- `ConcurrentBuilder` — fan-out to 3 parallel specialist agents
- `.with_aggregator()` — callback that combines results before synthesis
- `Agent` — each specialist is a standalone, independently deployable agent

> Note: `HandoffBuilder` is mentioned in slides for conceptual clarity. In code, convergence
> is via `.with_aggregator()` on `ConcurrentBuilder`. Both are valid MAF patterns.

## Framework

**Microsoft Agent Framework (MAF)** — unified successor to AutoGen + Semantic Kernel.
- Installed: `agent-framework==1.0.0rc5`
- Do NOT use AutoGen or Semantic Kernel directly — both deprecated in favor of MAF

## Azure Resources

- **Hub:** `devrel-aim-resource` (Microsoft.CognitiveServices/accounts, kind: AIServices, eastus2)
- **Project:** `maf-triage` (fresh project with maf-demo-insights linked)
- **Model:** `gpt-5-nano` deployment on `devrel-aim-resource`
- **Endpoint:** `https://devrel-aim-resource.cognitiveservices.azure.com/`
- **API version:** `2024-12-01-preview`
- **App Insights:** `maf-demo-insights` (linked to maf-triage project)

## Observability

### Working
- New Relic OTLP export (spans + root `triage_pipeline` span)
- New Relic Python APM agent (`newrelic.ini`, `run_demo.sh`)

### Partially working
- Azure Monitor / Foundry portal tracing — spans reach App Insights but
  Foundry Tracing tab not yet showing them (portal action needed: connect
  `maf-demo-insights` to `maf-triage` project in Foundry portal)

### Required env vars for full span content
```
ENABLE_SENSITIVE_DATA=true
AZURE_TRACING_GEN_AI_CONTENT_RECORDING_ENABLED=true
```

### Foundry Tracing investigation
See `docs/foundry-tracing-investigation.md` for full root cause analysis.

## Running the Demo

```bash
# CLI
source .venv/bin/activate
python triage.py

# Streamlit (streamlit-demo branch)
streamlit run app.py

# With New Relic APM
./run_demo.sh
```

SSL bypass required on corporate proxy:
```
AZURE_OPENAI_DISABLE_SSL=true
```

## Key Reference Links

- MAF intro blog: https://devblogs.microsoft.com/foundry/introducing-microsoft-agent-framework-the-open-source-engine-for-agentic-ai-apps/
- MAF RC blog: https://devblogs.microsoft.com/foundry/microsoft-agent-framework-reaches-release-candidate/
- MAF GitHub: https://github.com/microsoft/agent-framework
- Agent Framework Samples: https://github.com/microsoft/Agent-Framework-Samples
- Foundry tracing docs: https://learn.microsoft.com/en-us/semantic-kernel/concepts/enterprise-readiness/observability/telemetry-with-azure-ai-foundry-tracing

## Real-World Proof Points

- Microsoft TRIANGLE system: 97% triage accuracy, 600M logs/day
- BMW: 12x faster test-fleet data analysis using MAF + Foundry

## Branches

- `main` — core pipeline (triage.py, tools.py, agents/)
- `streamlit-demo` — adds app.py, newrelic.ini, run_demo.sh

## What NOT to Do

- Do not use AutoGen imports or patterns
- Do not use Semantic Kernel directly
- Do not use LangChain or LangGraph
- Do not hardcode API keys
- Do not use `pip` — always use `uv pip`
- Do not add complexity beyond demo needs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zmrfzn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-10 -->
