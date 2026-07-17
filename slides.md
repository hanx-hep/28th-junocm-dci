---
colorSchema: light
color: orange-light
layout: cover
routerMode: hash
theme: neversink
lineNumbers: true
favicon: /images/juno_logo_transparent.png
title: JUNO DCI monitoring — from dashboards to diagnosis
titleTemplate: '%s - Xiao Han'
---

<!-- transition: slide-up -->

<img src="/images/juno_logo_transparent.png" class="absolute right-16 top-14 w-28 opacity-80" />

# From dashboards to diagnosis

## Monitoring upgrades for the JUNO DCI

**Xiao Han** on behalf of the DCI Group<br/>
<a href="mailto:hanx@ihep.ac.cn"><Email v="hanx@ihep.ac.cn" /></a>

**28th JUNO Collaboration Meeting · 20 July 2026 · Beijing IHEP**

<a href="https://github.com/hanx-hep/28th-junocm-dci" class="ns-c-iconlink"><mdi-github /> Slides</a>
 · <a href="https://dci-grafana.ihep.ac.cn/" class="ns-c-iconlink"><mdi-view-dashboard-outline /> DCI Grafana</a>
 · <a href="https://hanx-hep.github.io/27th-junocm-dci/" class="ns-c-iconlink"><mdi-history /> 27th report</a>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# The upgrade is a shorter path from signal to action

:: content ::

<div class="lead text-center mt-3">
Monitoring is becoming an <strong>operational system</strong>, not only a collection of dashboards.
</div>

<div class="three-cards mt-8">
  <div class="story-card">
    <mdi-source-branch class="story-icon" />
    <h2>Reproducible</h2>
    <p>Dashboard JSON and provisioning live in Git, so changes can be reviewed and deployments reconstructed.</p>
  </div>
  <div class="story-card">
    <mdi-layers-search class="story-icon" />
    <h2>Diagnosable</h2>
    <p>Metrics show the symptom; centralized component logs provide the event-level context behind it.</p>
  </div>
  <div class="story-card">
    <mdi-robot-outline class="story-icon" />
    <h2>Accessible</h2>
    <p>Operators use Grafana directly; agents reach the same evidence through the controlled IHEP MCP gateway.</p>
  </div>
</div>

<div class="takeaway mt-6">
<strong>Goal:</strong> reduce the time between “something is wrong” and “this is the next useful action.”
</div>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# One monitoring loop, three upgrades

:: content ::

```mermaid {scale: 0.56}
flowchart LR
    subgraph S[Operational signals]
        M[Service and host metrics]
        T[TPC transfer records]
        L[DIRAC component logs]
    end

    subgraph E[Central evidence]
        P[(Prometheus)]
        Q[(Elasticsearch)]
    end

    R[Dashboard JSON in Git]
    G[DCI Grafana]
    H[Operators]
    C[AI agent / MCP client]
    W[mcp.ihep.ac.cn]
    X[mcp-grafana]

    M --> P
    T --> Q
    L --> A[ActiveMQ] --> K[Logstash] --> Q
    P --> G
    Q --> G
    R -->|provision| G
    G -->|visual evidence| H
    C -->|authenticated request| W --> X -->|Grafana API / render| G
```

<div class="architecture-legend">
  <span><strong>Foundation</strong> · Git + provisioning</span>
  <span><strong>Evidence</strong> · Prometheus + Elasticsearch</span>
  <span><strong>Interfaces</strong> · Grafana + MCP</span>
</div>

---
layout: section
color: cyan-light
---

# 1 · Make dashboards reproducible

---
layout: top-title-two-cols
color: gray-light
align: c-l-l
---

:: title ::

# Provisioning moves the control point into Git

:: left ::

## Before · instance state

```mermaid {scale: 0.46}
flowchart TD
    A[Edit in Grafana UI] --> B[State in running instance]
    B --> C[Manual backup]
    C --> D[Recovery after change]
```

The running service was the main source of truth. A backup could recover state, but it did not make each change easy to review, reproduce, or transfer.

:: right ::

## Now · delivery path

```mermaid {scale: 0.46}
flowchart TD
    A[Dashboard JSON] --> B[Git diff and review]
    B --> C[Provisioning provider]
    C --> D[Grafana instance]
```

The repository becomes the durable definition of the monitoring layout. The instance reconciles that definition on deployment and during provider refresh.

<div class="takeaway compact mt-5">
Backup protects the past. Provisioning controls the next change.
</div>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# The repository is now an operational control surface

:: content ::

<div class="provider-grid mt-4">
  <div class="provider"><strong>10</strong><span>Admin</span></div>
  <div class="provider"><strong>9</strong><span>DIRAC</span></div>
  <div class="provider"><strong>6</strong><span>TPC</span></div>
  <div class="provider"><strong>4</strong><span>User</span></div>
  <div class="provider"><strong>2</strong><span>Shift</span></div>
</div>

<div class="delivery-loop mt-8">
  <div><small>CREATE</small><strong>UI or agent</strong></div>
  <mdi-arrow-right />
  <div><small>CAPTURE</small><strong>Dashboard JSON</strong></div>
  <mdi-arrow-right />
  <div><small>CONTROL</small><strong>Git review</strong></div>
  <mdi-arrow-right />
  <div><small>RECONCILE</small><strong>30 s refresh</strong></div>
</div>

<div class="two-notes mt-8">
  <div><strong>31 dashboard files</strong><br/>Five providers reconstruct the current folder layout from version-controlled JSON.</div>
  <div class="warning-note"><strong>Guard against drift</strong><br/><code>allowUiUpdates: true</code> keeps UI editing convenient; export → review → commit must remain the return path to Git.</div>
</div>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# TPC transfer matrix: failures become patterns

:: content ::

<img src="/images/tpc-transfer-matrix-2026-07-17.png" class="matrix-shot mt-2" />

<div class="matrix-caption">
  <span><strong>4 modes</strong> · pull / push / streamed / all</span>
  <span><strong>12 panels</strong> · 8 tables + 4 state timelines</span>
  <span><strong>5 variables</strong> · time, sites, state, mode</span>
</div>

<div class="takeaway compact matrix-takeaway mt-2">
A grid turns individual transfer results into site-, direction-, and mode-correlated failure patterns. <span class="muted">Snapshot: 17 July 2026 · last 7 days.</span>
</div>

---
layout: top-title
color: green-light
align: c
---

:: title ::

# AI assistance belongs inside the review loop

:: content ::

```mermaid {scale: 0.65}
flowchart LR
    A[Operator defines<br/>semantics and thresholds] --> B[Agent composes<br/>dashboard JSON]
    B --> C[Build, diff<br/>and review]
    C --> D[Provision to<br/>Grafana]
    D --> E[Operator verifies<br/>the operational view]
    E -. feedback .-> A
```

<div class="two-notes mt-8">
  <div><strong>Good use of automation</strong><br/>Repeat panel structure, queries, transformations, variables, and layout consistently across a test matrix.</div>
  <div><strong>Human control remains explicit</strong><br/>Domain meaning, grading thresholds, acceptance, and operational action stay with the operator.</div>
</div>

<div class="takeaway mt-8">
The benefit is not “AI made a dashboard.” It is <strong>faster composition with a normal Git review boundary</strong>.
</div>

---
layout: section
color: purple-light
---

# 2 · Give agents controlled access

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# MCP adds a controlled machine interface to Grafana

:: content ::

```mermaid {scale: 0.54}
flowchart LR
    C[AI agent or MCP client] -->|MCP JSON-RPC| G[mcp.ihep.ac.cn<br/>central gateway]

    subgraph CP[Central identity and policy]
        G -->|verify key and scope| A[Auth service]
        A --> L[(IHEP LDAP)]
        A --> P[(PostgreSQL)]
    end

    G -->|authorized tool call| M[mcp-grafana]

    subgraph MB[Monitoring boundary]
        M -->|Grafana API / render| F[DCI Grafana]
        F --> D[(Prometheus / Elasticsearch)]
        F --> R[grafana-image-renderer]
        R --> F
    end
```

<div class="two-notes compact-notes mt-2">
  <div><strong>Policy stays centralized</strong><br/>One authenticated gateway enforces identity and scope.</div>
  <div><strong>Grafana stays behind the boundary</strong><br/><code>mcp-grafana</code> adapts dashboard metadata, queries, and rendering.</div>
</div>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# From an operational question to evidence

:: content ::

<div class="question-flow mt-6">
  <div class="flow-step"><small>1 · ASK</small><strong>“Why is TPC push failing for a site pair?”</strong></div>
  <mdi-arrow-right />
  <div class="flow-step"><small>2 · AUTHORIZE</small><strong>Gateway verifies key and scope</strong></div>
  <mdi-arrow-right />
  <div class="flow-step"><small>3 · INSPECT</small><strong>Query data and render the relevant panel</strong></div>
  <mdi-arrow-right />
  <div class="flow-step"><small>4 · EXPLAIN</small><strong>Return evidence and the next diagnostic step</strong></div>
</div>

<div class="evidence-grid mt-10">
  <div><mdi-code-json /><strong>Structured evidence</strong><span>dashboard definitions, variables, queries, and data results</span></div>
  <div><mdi-image-search-outline /><strong>Visual evidence</strong><span>rendered panels preserve the pattern an operator would see</span></div>
  <div><mdi-shield-account-outline /><strong>Operational boundary</strong><span>the agent gathers and explains; remediation remains an explicit action</span></div>
</div>

<div class="takeaway mt-9">
MCP is an access path to monitoring evidence — <strong>not another monitoring data source</strong>.
</div>

---
layout: section
color: lime-light
---

# 3 · Put logs beside metrics

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# Central logs close the context gap

:: content ::

```mermaid {scale: 0.70}
flowchart LR
    A[DIRAC components] -->|publish log messages| B[ActiveMQ]
    B -->|consume and parse| C[Logstash]
    C -->|index| D[(Elasticsearch)]
    D -->|query and aggregate| E[Grafana]
    E --> F[Component Logs dashboard]
```

<div class="question-cards mt-8">
  <div><small>METRICS</small><strong>What changed?</strong><span>Resource and service behavior reveal the symptom.</span></div>
  <div><small>TIMELINE</small><strong>When did it start?</strong><span>Central timestamps define the relevant diagnostic window.</span></div>
  <div><small>LOG RECORDS</small><strong>Which component explains it?</strong><span>Event-level context replaces host-by-host inspection.</span></div>
</div>

<div class="takeaway mt-8">
Prometheus and Elasticsearch answer different questions; Grafana brings both into the same operational workflow.
</div>

---
layout: top-title
color: gray-light
align: c
---

:: title ::

# The Component Logs dashboard follows the triage sequence

:: content ::

<div class="log-panels mt-3">
  <div class="log-panel">
    <div class="mock-pie"></div>
    <div><small>1 · DISTRIBUTION</small><h2>Is the error mix abnormal?</h2><p>One pie panel compares information, warning, and error volume.</p></div>
  </div>
  <div class="log-panel">
    <svg class="mock-line" viewBox="0 0 240 90" aria-label="example log volume timeline"><polyline points="0,70 25,66 50,68 75,42 100,54 125,20 150,35 175,28 200,60 240,48" /></svg>
    <div><small>2 · TIMELINE</small><h2>When did the change begin?</h2><p>One time-series panel narrows the investigation window.</p></div>
  </div>
  <div class="log-panel">
    <div class="mock-table"><i></i><i></i><i></i><i></i></div>
    <div><small>3 · RECORDS</small><h2>Which message is actionable?</h2><p>One table exposes the individual records for investigation.</p></div>
  </div>
</div>

<div class="filter-strip mt-5">
  <strong>Filter the same evidence by</strong>
  <span>Category</span><span>Name</span><span>Level</span>
  <a href="https://dci-grafana.ihep.ac.cn/d/bfgu666p30xdsb/component-logs?orgId=1&from=now-24h&to=now"><mdi-open-in-new /> Open live dashboard</a>
</div>

---
layout: top-title
color: green-light
align: c
---

:: title ::

# The operational loop is now connected

:: content ::

```mermaid {scale: 0.66}
flowchart LR
    S[TPC or service symptom] --> V[Grafana operational view]
    V --> E[Query metrics and logs]
    E --> D[Evidence-based diagnosis]
    D --> A[Operator action]
    A -. learning .-> C[Dashboard / alert as code]
    C -. provision .-> V
    M[MCP agent] -. gather and explain .-> E
```

<div class="role-grid mt-9">
  <div><strong>Grafana</strong><span>makes the pattern visible</span></div>
  <div><strong>Metrics + logs</strong><span>provide complementary evidence</span></div>
  <div><strong>MCP agent</strong><span>shortens evidence gathering</span></div>
  <div><strong>Operator</strong><span>owns judgment and action</span></div>
</div>

<div class="takeaway mt-8">
The architecture connects observation to diagnosis while keeping change review and operational authority explicit.
</div>

---
layout: top-title-two-cols
color: gray-light
align: c-l-l
---

:: title ::

# Delivered now, with a clear next increment

:: left ::

## Delivered

<div class="status-stack">
  <div><mdi-check-circle-outline /><span><strong>Dashboard provisioning</strong><br/>31 JSON dashboards under five providers</span></div>
  <div><mdi-check-circle-outline /><span><strong>TPC transfer view</strong><br/>12 panels across four transfer modes</span></div>
  <div><mdi-check-circle-outline /><span><strong>Controlled MCP path</strong><br/>Central gateway to <code>mcp-grafana</code></span></div>
  <div><mdi-check-circle-outline /><span><strong>Central component logs</strong><br/>Three complementary diagnostic panels</span></div>
</div>

:: right ::

## Next increment

<div class="status-stack next">
  <div><mdi-arrow-right-circle-outline /><span><strong>Actionable alerts</strong><br/>Thresholds, ownership, and response links</span></div>
  <div><mdi-arrow-right-circle-outline /><span><strong>Dashboard release checks</strong><br/>Build, schema, and screenshot smoke tests</span></div>
  <div><mdi-arrow-right-circle-outline /><span><strong>Scoped MCP operations</strong><br/>Read-first tools with explicit authorization</span></div>
  <div><mdi-arrow-right-circle-outline /><span><strong>Metrics-to-logs drill-down</strong><br/>Carry site, component, and time context</span></div>
</div>

---
layout: top-title
color: green-light
align: c
---

:: title ::

# Three takeaways

:: content ::

<div class="three-cards takeaway-cards mt-8">
  <div class="story-card"><strong>1</strong><h2>Reproducible</h2><p>Git and provisioning turn dashboard changes into reviewable, reconstructable configuration.</p></div>
  <div class="story-card"><strong>2</strong><h2>Diagnosable</h2><p>TPC views expose patterns; centralized logs explain the component events behind them.</p></div>
  <div class="story-card"><strong>3</strong><h2>Accessible</h2><p>The same Grafana evidence serves operators directly and agents through the IHEP MCP gateway.</p></div>
</div>

<div class="closing-line mt-12">
The key upgrade is not another dashboard.<br/>
It is a <strong>shorter, controlled path from signal to action</strong>.
</div>

---
layout: cover
color: navy
loop: true
title: Questions
---

# Questions?

## Thank you

**Xiao Han · IHEP, CC**<br/>
DCI Group · JUNO Collaboration

<a href="https://dci-grafana.ihep.ac.cn/" class="ns-c-iconlink"><mdi-view-dashboard-outline /> dci-grafana.ihep.ac.cn</a>
 · <a href="https://github.com/hanx-hep/28th-junocm-dci" class="ns-c-iconlink"><mdi-github /> slides & source</a>
