# From Dashboards to Diagnosis

## 15-minute English speaking script

The timings are approximate and include short pauses for the two live Grafana views. Total planned time is about 15 minutes.

## Slide 1 — From dashboards to diagnosis · 0:35

Good afternoon, everyone. I am Xiao Han, presenting on behalf of the DCI Group. Today I will report the latest monitoring upgrades for the JUNO distributed computing infrastructure. The title, “From dashboards to diagnosis,” summarizes the direction of this work. We are not only adding more charts. We are connecting configuration, monitoring evidence, and controlled agent access so that an operational symptom can lead more quickly to a useful diagnosis and a clear next action.

## Slide 2 — A shorter path from signal to action · 0:55

The main message is shown on this slide. Monitoring is becoming an operational system rather than a collection of independent dashboards. We are improving three properties. First, it is reproducible: dashboard definitions and provisioning are kept in Git. Second, it is diagnosable: metrics show the symptom, while centralized logs explain the events behind it. Third, it is accessible: operators use Grafana directly, and agents can reach the same evidence through the controlled IHEP MCP gateway. The goal is simple—to shorten the time between noticing that something is wrong and knowing the next useful action.

## Slide 3 — One monitoring loop · 1:10

This diagram gives the complete picture. On the left are the operational signals: service and host metrics, TPC transfer records, and DIRAC component logs. Metrics go to Prometheus. Transfer records and centralized logs are available through Elasticsearch, with component logs passing through ActiveMQ and Logstash. Grafana brings these data sources together as the operational view. Dashboard JSON in Git is provisioned into Grafana, so the visual layer is also controlled configuration. Operators inspect Grafana directly. An AI agent reaches Grafana through the centralized MCP gateway and the mcp-grafana adapter. These are the three upgrades I will discuss: a reproducible foundation, richer evidence, and controlled interfaces.

## Slide 4 — Make dashboards reproducible · 0:15

I will begin with the foundation: making dashboards reproducible. This sounds like a configuration detail, but it changes how safely we can review, deploy, and recover the monitoring environment.

## Slide 5 — Provisioning moves the control point · 1:05

Previously, the running Grafana instance was effectively the main source of truth. We edited dashboards in the UI, Grafana stored the state, and a manual backup protected us if recovery was needed. A backup is useful, but it does not give a clear review path for every change. In the current workflow, dashboard JSON is stored in Git, reviewed as a normal diff, and loaded through a provisioning provider. This makes the monitoring layout reconstructable and makes changes visible before deployment. The distinction at the bottom is important: backup protects the past, while provisioning controls the next change.

## Slide 6 — The repository as a control surface · 1:10

The repository now contains 31 dashboard files under five providers: Admin, DIRAC, TPC, User, and Shift. The delivery loop is create, capture, control, and reconcile. An operator or an agent can compose a dashboard, but the result must return as dashboard JSON. Git is the review boundary, and Grafana refreshes the provider every 30 seconds. One guardrail deserves attention: UI updates are still allowed for convenience. Therefore, an edit made in Grafana must be exported, reviewed, and committed. Otherwise, the running instance can drift away from the repository. Git remains the durable source of truth.

## Slide 7 — Live TPC transfer matrix · 1:20

This is the live TPC transfer monitoring dashboard. It may take a moment to load. At the top, we can filter by time interval, source site, destination site, success state, and copy mode. The dashboard contains 12 panels: eight tables and four state timelines, covering pull, push, streamed, and combined modes.

[Demo: scroll vertically through the matrix and compare at least two copy modes.]

The value of the matrix is correlation. A single failed transfer is only an event. A repeated row, column, or mode pattern suggests that the problem follows a site, a direction, or a transfer method. The average panels also help distinguish a transient failure from persistent degradation. This turns many individual test records into a compact operational signal.

## Slide 8 — AI inside the review loop · 0:55

The TPC dashboard also demonstrates where AI assistance is useful. The operator defines the semantics, the transfer modes, the thresholds, and what the colors mean. The agent can then handle repetitive dashboard composition: panel structure, queries, transformations, variables, and layout. The output is built, diffed, and reviewed before provisioning. Finally, the operator verifies the operational view. So the important result is not that AI made a dashboard. The result is faster composition while preserving a normal Git review boundary and explicit human control.

## Slide 9 — Controlled agent access · 0:15

The second upgrade is controlled machine access. Once monitoring evidence is structured, an agent can use it—but only through a clear authentication and authorization boundary.

## Slide 10 — MCP-Grafana architecture · 1:15

The client does not connect directly to Grafana. It sends an MCP JSON-RPC request to the centralized gateway at mcp.ihep.ac.cn. The gateway verifies the key and scope through the authentication service, which is connected to IHEP LDAP and PostgreSQL. Only an authorized tool call is forwarded to mcp-grafana. Inside the monitoring boundary, mcp-grafana uses the Grafana API and rendering interface. Grafana reads Prometheus and Elasticsearch and can use the image renderer for visual context. This separation gives us two useful properties: identity and policy stay centralized, while Grafana-specific behavior stays inside a dedicated adapter.

## Slide 11 — From question to evidence · 1:00

Here is a concrete diagnostic path. The user asks why TPC push transfers are failing for a site pair. The gateway first verifies the caller and scope. Then mcp-grafana queries the relevant data and can render the panel that an operator would inspect. The agent returns the evidence and proposes the next diagnostic step. There are two forms of evidence: structured definitions and query results, and visual patterns from rendered panels. The boundary is equally important. The agent gathers and explains evidence; remediation remains an explicit operational action. MCP is an access path, not a new monitoring data source.

## Slide 12 — Put logs beside metrics · 0:15

The third upgrade is to put centralized component logs beside metrics. Metrics are good at showing that behavior changed. Logs are needed to explain which component event caused that change.

## Slide 13 — Central logs close the gap · 1:00

DIRAC components publish log messages to ActiveMQ. Logstash consumes and parses them, indexes them in Elasticsearch, and Grafana provides the query and visualization layer. This pipeline supports three stages of triage. Metrics answer, “What changed?” The timeline answers, “When did it begin?” Individual records answer, “Which component explains it?” Central timestamps and common filters define one investigation window, replacing the need to visit service hosts one by one. Prometheus and Elasticsearch answer different questions, but Grafana puts both in the same operational workflow.

## Slide 14 — Live Component Logs view · 1:15

This is the live Component Logs dashboard. It loads more slowly than the TPC dashboard, so I will give it a moment. The vertical pane on the right is scaled down, and we can scroll inside it.

[Demo: scroll through the three panels when they have loaded.]

The first panel shows the distribution of information, warning, and error messages. The second shows how the volume changes over time. The third exposes individual records. A practical investigation starts broad and then narrows: first identify an abnormal error mix, then select the time window, and finally inspect the responsible message. Category, Name, and Level filters keep the same context across all three views.

## Slide 15 — The connected operational loop · 1:00

These upgrades now form one connected loop. A TPC or service symptom appears in Grafana. We query the relevant metrics and logs, build an evidence-based diagnosis, and leave the operational action with the operator. The MCP agent can shorten evidence gathering and explanation, but it does not remove human judgment. After the incident, the learning can return to the system as a better dashboard or alert definition. Because that definition is code, it passes through review and provisioning before becoming part of the next operational view.

## Slide 16 — Delivered and next · 1:00

The left side summarizes what is delivered now: dashboard provisioning with 31 JSON files, the 12-panel TPC transfer view, the controlled MCP path to Grafana, and centralized component logs with three complementary views. The right side shows the next increment. We need actionable alerts with clear ownership, automated dashboard release checks, read-first MCP operations with explicit authorization, and drill-down links that carry site, component, and time context from metrics to logs. These steps turn the current architecture into a more repeatable operational practice.

## Slide 17 — Three takeaways · 0:50

There are three takeaways. First, the monitoring environment is more reproducible because dashboards are reviewed and provisioned from Git. Second, it is more diagnosable because TPC views expose patterns and centralized logs provide component-level context. Third, it is more accessible because the same Grafana evidence serves operators directly and agents through the IHEP MCP gateway. The key upgrade is not simply another dashboard. It is a shorter and controlled path from signal to action.

## Slide 18 — Questions · 0:15

That concludes my update. Thank you for your attention, and I am happy to take questions.
