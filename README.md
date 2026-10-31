# Cloud Attack Path Visualizer using Graph-Based Permission Modelling

> **SAFE SIMULATION GUARANTEE**: This project operates strictly on local, simulated JSON permission data. It **does NOT** connect to AWS, Azure, GCP, or any real cloud infrastructure, and does **NOT** execute real attacks or exploits.

---

## 1. Project Overview

Modern cloud infrastructure relies on complex access control mechanisms (identities, roles, attached policies, trust relationships, wildcard permissions, resource sensitivity). These relationships can combine to create hidden **privilege-escalation attack paths** where low-privileged identities can gain administrative privileges or access sensitive resources.

The **Cloud Attack Path Visualizer** models cloud environments as directed access control graphs, discovers attack paths using bounded BFS/DFS graph algorithms, evaluates risk using explainable multi-factor scoring, renders interactive visual attack chains in React Flow, and generates printable executive security reports.

---

## 2. Technology Stack

- **Frontend**: React 18, TypeScript, Vite, Tailwind CSS, React Flow (`@xyflow/react`), Recharts, Lucide Icons.
- **Backend**: Python 3.12, FastAPI, Pydantic v2, NetworkX (Graph Engine), Uvicorn.
- **Testing**: Pytest for backend unit and API testing.
- **Data & Reports**: JSON environment schemas, dynamic HTML report generator with printable PDF CSS layout.

---

## 3. System Architecture & Workflow

```
 [ Simulated JSON ]
        │
        ▼
 ┌────────────────────────────────────────────────────────┐
 │                   Graph Builder Engine                 │
 │ (NetworkX DiGraph: Users, Roles, Policies, Resources)  │
 └──────────────────────────┬─────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        ▼                                       ▼
 ┌──────────────────────────┐       ┌──────────────────────────┐
 │   Attack Path Analyzer   │       │       Risk Engine        │
 │  (BFS/DFS Bounded Paths) │       │ (Explainable 0-100 Score)│
 └─────────────┬────────────┘       └───────────┬──────────────┘
               │                                │
               └────────────────────┬───────────┘
                                    ▼
 ┌────────────────────────────────────────────────────────┐
 │                 React Flow Visualizer                  │
 │   (Animated Path Highlighting, Node Inspector Drawer)  │
 └────────────────────────────────────────────────────────┘
```

---

## 4. Graph Model & Edge Metadata

The environment JSON is parsed into a directed graph $G = (V, E)$:

### Node Types ($V$)
- `USER`: Identity objects (`privilege`: low, medium, high, admin).
- `GROUP`: Identity groups.
- `ROLE`: Access roles (`privilege`: low, medium, high, admin).
- `POLICY`: Permission documents (`actions`, `resources`, `effect`).
- `RESOURCE`: Assets (`type`: database, storage, secret, compute; `sensitivity`: low, medium, high, critical).
- `ADMIN_PRIVILEGE`: Root administrative targets.

### Relationship Edges ($E$)
- `USER` $\rightarrow$ `ROLE` (`has_role`)
- `USER` $\rightarrow$ `GROUP` (`member_of`)
- `GROUP` $\rightarrow$ `ROLE` (`has_role`)
- `ROLE` $\rightarrow$ `POLICY` (`attached_policy`)
- `POLICY` $\rightarrow$ `RESOURCE` (`grants_permission`)
- `ROLE` $\rightarrow$ `ROLE` (`can_assume` via `sts:AssumeRole`)
- `ROLE` $\rightarrow$ `ADMIN_PRIVILEGE` (`escalates_to_admin`)

---

## 5. Explainable Risk Scoring Engine

The risk score $R \in [0, 100]$ is computed deterministically:

$$\text{Risk Score} = S_{\text{escalation}} + S_{\text{sensitivity}} + S_{\text{dangerous\_perm}} + S_{\text{chain\_length}}$$

1. **Privilege Escalation Jump ($0-35$)**: Low/Medium user reaching Admin role/console (+35).
2. **Target Sensitivity ($0-30$)**: `critical` (+30), `high` (+20), `medium` (+10).
3. **Dangerous Permissions ($0-25$)**: Wildcards `*` (+25), `sts:AssumeRole` (+15), `iam:PassRole` (+15).
4. **Chain Complexity ($0-10$)**: Direct access ($\le 2$ hops, +10), multi-hop transition (+5).

### Severity Levels
- **0 – 24**: LOW
- **25 – 49**: MEDIUM
- **50 – 74**: HIGH
- **75 – 100**: CRITICAL

---

## 6. Quickstart: Running Locally

### Prerequisites
- Python 3.12+
- Node.js v20+ / npm v10+

### Step 1: Start Backend (FastAPI)
```bash
cd backend
python -m venv venv

# On Windows:
.\venv\Scripts\pip install -r requirements.txt
.\venv\Scripts\python -m app.main

# On Linux/macOS:
source venv/bin/activate
pip install -r requirements.txt
python -m app.main
```
Backend API will start at `http://localhost:8000`.

### Step 2: Start Frontend (Vite + React)
```bash
cd frontend
npm install
npm run dev
```
Frontend web application will start at `http://localhost:5173`.

---

## 7. Running Backend Unit Tests

```bash
cd backend
.\venv\Scripts\pytest
```

---

## 8. Sample Datasets Included

1. `sample-data/basic-environment.json`: Baseline environment with standard users, roles, and low-sensitivity resources.
2. `sample-data/privilege-escalation.json`: Low-priv user `alice` escalating to `AdminRole` via `sts:AssumeRole` on `FinanceRole`.
3. `sample-data/critical-risk.json`: Wildcard permission (`*`) on critical secret store & production compute node.
4. `sample-data/multi-path.json`: User `bob` with 3 separate paths of varying risk levels to `Customer Vault DynamoDB`.

---

## 9. REST API Endpoints

- `GET /api/health`: Health status.
- `GET /api/environment`: Get active cloud environment.
- `POST /api/environment/load`: Load custom environment JSON.
- `POST /api/environment/preset/{preset_name}`: Load preset scenario.
- `POST /api/environment/entity/{type}`: Create user, role, policy, or resource.
- `DELETE /api/environment/entity/{id}`: Delete entity.
- `POST /api/environment/relationship`: Add relationship edge.
- `DELETE /api/environment/relationship/{id}`: Delete relationship edge.
- `GET /api/graph`: Get React Flow layout graph.
- `GET /api/graph/node/{node_id}`: Node attribute and connection inspector.
- `POST /api/analyze`: Run attack path analysis.
- `GET /api/findings`: List security findings.
- `POST /api/report`: Generate dynamic HTML security report.
- `GET /api/statistics`: Dashboard KPIs and charts data.

---

## 10. Limitations & Future Scope

### Current Limitations
- Operates on static JSON models (no live cloud credentials).
- Graph layout uses deterministic column grouping.

### Future Scope
- **Live AWS IAM / Azure RBAC / GCP IAM Ingestion**: Direct cloud API connectors.
- **Multi-Cloud Cross-Tenant Graph Analysis**: Cross-account trust chain discovery.
- **Automated Remediation**: Safe terraform/cloudformation fix generation.
