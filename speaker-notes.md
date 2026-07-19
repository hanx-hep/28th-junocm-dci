# Monitoring Upgrades for the JUNO DCI

## 15-minute English speaking script

The timings are approximate. They include short pauses for the two live Grafana views. The full talk should take about 15 minutes.

## Slide 1 — Monitoring upgrades for the JUNO DCI · 0:30

Good afternoon. I am Xiao Han, and I am speaking on behalf of the DCI Group. Today I will present our recent work on monitoring for JUNO distributed computing. This report covers dashboard configuration in Git, central component logs, access to Grafana through MCP, and the use of an AI agent in the dashboard workflow.

## Slide 2 — Overview · 0:55

Here is the overview. Monitoring now does more than display charts. It supports JUNO data reprocessing. It helps us raise alerts, locate faults, and recover services. We made three technical updates. Dashboard JSON is stored in Git. Component logs are collected in one place. Agents can access Grafana through MCP. I will then show how AI can support the complete dashboard workflow.

## Slide 3 — Make dashboards reproducible · 0:10

I will start with dashboard configuration. The goal is to store dashboard definitions in Git and load them into Grafana in a standard way.

## Slide 4 — Dashboards are stored in a Git repository · 1:00

The left side shows the old method. We edited a dashboard in the Grafana UI. Grafana stored the result in its database. We made a backup when needed. This worked for recovery, but it did not provide a review process for each change. The right side shows the current method. We can still edit a dashboard in the UI, or edit its JSON directly. We then review the JSON diff in Git. The provisioning provider loads the reviewed file into Grafana. Git now stores the dashboard definition.

## Slide 5 — Dashboard files and the delivery loop · 1:05

The repository now has 31 dashboard files. They are grouped under five providers: Admin, DIRAC, TPC, User, and Shift. The workflow has four steps. We create a dashboard, capture it as JSON, review it in Git, and let Grafana load it. Grafana checks the provider every 30 seconds. UI editing is still enabled. So an edit made in Grafana must come back to Git. We export it, review it, and commit it. Otherwise, Grafana and the repository can contain different versions.

## Slide 6 — Panel configuration details · 0:50

This slide shows the main configuration steps. First, we register a file-based provisioning provider. It points to the TPC dashboard directory and checks for updates every 30 seconds. Second, we mount the provisioning directory into the Grafana container. Third, we export the existing dashboards from grafana.db and save them as JSON files. After these steps, Grafana can load the dashboard definitions from the repository.

## Slide 7 — Save component logs to a database · 0:10

The second part is component logs. Metrics show that a service changed. Logs help us find the component event related to that change.

## Slide 8 — Central component logs · 1:05

JUNO DIRAC runs more than 60 components. By default, each component writes logs to its local disk. In CAS_Prod.cfg, we keep stdout and add mqLogs for both services and agents. The mqLogs backend sends the messages to ActiveMQ. Logstash reads and parses them, then writes them to Elasticsearch. Grafana queries this central store. This configuration gives services and agents the same log path while keeping local output available.

## Slide 9 — Component Logs dashboard · 1:10

This is the live Component Logs dashboard. It may load slowly, so I will wait for a moment. The left side shows the three investigation steps. The live dashboard is on the right, and we can scroll inside it.

[Demo: wait for the dashboard, then scroll through the three panels.]

The first panel shows the number of information, warning, and error messages. The second panel shows how that number changes over time. The third panel shows individual records. We first check the message levels. Then we select a time window. Finally, we read the related records. The Category, Name, and Level filters apply to all three panels.

## Slide 10 — Give agents controlled access · 0:10

The third part is MCP access. An agent can use monitoring data, but it must pass through authentication and authorization checks.

## Slide 11 — MCP access to Grafana · 1:10

The client does not connect to Grafana directly. It sends an MCP request to mcp.ihep.ac.cn. This is the central gateway. The gateway checks the API key and its scope. The authentication service uses IHEP LDAP and PostgreSQL. If the request is allowed, the gateway sends the tool call to mcp-grafana. This service uses the Grafana API and the rendering interface. Grafana reads data from Prometheus and Elasticsearch. In this design, identity and policy stay at the central gateway. Grafana operations stay in the mcp-grafana service.

## Slide 12 — From an operational question to evidence · 1:00

Here is one example. A user asks why TPC push transfers are failing for a site pair. First, the gateway checks the user and the requested scope. Then the agent can use search_dashboards and get_dashboard_summary to find the dashboard. It can read panel queries with get_dashboard_panel_queries and query Prometheus when needed. It can return a panel image with get_panel_image or a Grafana link with generate_deeplink. These are actual tools returned by the current IHEP MCP tools/list response. They support inspection and explanation. Write operations still need explicit authorization.

## Slide 13 — AI workflow · 0:10

The last part shows how an AI agent can support the dashboard workflow from creation to verification.

## Slide 14 — AI assistance throughout the monitoring workflow · 1:00

The operator starts by defining the meaning of the dashboard and its thresholds. The agent receives this request and creates the dashboard JSON. We build the result and review the Git diff. The reviewed file is then provisioned to Grafana. The agent can check the result through the MCP server. If a problem is found, the result goes back to the agent for another change. The agent handles repeated work. The operator still defines the meaning, approves the result, and decides what action to take.

## Slide 15 — TPC transfer matrix · 1:15

This is the live TPC transfer matrix. It is based on Albert Dzakhoev's work. We used AI to improve the sorting and add panels for the average success rate. The dashboard covers pull, push, streamed, and combined transfer modes.

[Demo: scroll through the matrix and compare two transfer modes.]

A single failed transfer is one event. A repeated row or column can point to a site problem. A pattern in one mode can point to a transfer-method problem. The average success rate helps us see whether a problem is short or continues over time. The matrix gives us a summary of many transfer tests.

## Slide 16 — Delivered work and next steps · 1:00

The left side lists the work that is available now. We have 31 provisioned dashboard files, the TPC transfer view, MCP access to Grafana, and central component logs. The right side lists the next steps. We need alerts with owners and response links. We need checks for dashboard builds, schemas, and screenshots. MCP operations should start with read-only access and clear authorization. We also want links from metrics to logs that keep the site, component, and time range.

## Slide 17 — Summary · 0:50

I have three points to summarize. First, Git and provisioning let us review and restore dashboard definitions. Second, TPC views and component logs provide data for diagnosis. Third, operators and agents can use the same Grafana data through different access paths. Operators use Grafana directly. Agents use the IHEP MCP gateway. The main update is the workflow from a monitoring signal to an operator action.

## Slide 18 — Questions · 0:15

That is the end of my report. Thank you. I am ready to take questions.
