# 📊 GKE VPA Rightsizer Agent

An intelligent, multi-agent automated pipeline built with the **Google Agent Development Kit (ADK)**. It automates GKE cluster resource analysis, queries Vertical Pod Autoscaler (VPA) metrics, generates cleaned and updated Kubernetes manifests, compiles an interactive web dashboard, and deploys it to Google Cloud Run.

---

## 🚀 Key Features

* **Multi-Agent Pipeline Orchestration**: Implements sequential checks and verification between dedicated subagents to prevent failures and handle automated workflows.
* **Intelligent GKE Scraping (`gke_scraper`)**:
  * Scans multiple GKE clusters dynamically across Google Cloud project(s).
  * Extracts active Vertical Pod Autoscaler (VPA) targets, lower bounds, and upper bounds.
  * Dynamically queries **Cloud Monitoring fallback metrics** for workloads missing VPA definitions.
  * Prompts for missing Project IDs on the fly using the built-in `request_input` tool.
* **Dataset & Asset Builder (`web_builder`)**: Parses raw scraper findings into structured JSON and copies updated, clean deployment manifests.
* **Cloud Run Deployment (`cloud_run_deployer`)**: Deploys the Express-based interactive reporting dashboard to Google Cloud Run.

---

## 📂 Project Structure

```
├── agent.py                      # Main pipeline orchestrator (RootVpaPipeline)
├── agents-cli-manifest.yaml      # ADK app manifest file
├── .gitignore                    # Excludes cache, environments, and dynamic outputs
├── subagents/                    # Subagent definitions
│   ├── scraper_agent.py          # gke_scraper (gke scraper subagent)
│   ├── builder_agent.py          # web_builder (web builder subagent)
│   └── deployer_agent.py         # cloud_run_deployer (cloud run deployer subagent)
├── tools/                        # Python execution tools used by subagents
│   ├── scraper_tool.py           # Executes project-wide scans (calls scan_and_generate.py)
│   ├── builder_tool.py           # Natively parses findings into public vpa-data.json
│   └── deployer_tool.py          # Builds and deploys dashboard to Cloud Run
└── results/                      # Git-ignored local output folder
    ├── vpa_recommendations_report.md
    └── vpa-<project_id>/
        └── vpa-<cluster_name>/
            └── <namespace>/
                └── <deployment-name>.yaml
```

---

## 🛠️ Usage Instructions

### 1. Run the Pipeline Locally
To run the entire end-to-end scraper, dashboard builder, and Cloud Run deployment pipeline locally via the CLI:

```bash
# With explicit project ID(s)
agents-cli run --app-name vpa_rightsizer "Scan GKE clusters in project gkeop002, compile the dashboard, and deploy it to Cloud Run."

# Without project ID (scapers will prompt for input dynamically)
agents-cli run --app-name vpa_rightsizer "Scan GKE clusters, generate recommendations, and build the dashboard."
```

### 2. Deployment & Platform Publication
To publish this agent on the **Gemini Enterprise Agent Platform**:

#### Step A: Enhance and Configure Runtime Targets
```bash
agents-cli scaffold enhance . --deployment-target agent_runtime
```

#### Step B: Deploy to Vertex AI Agent Runtime
```bash
agents-cli deploy --project YOUR_PROJECT_ID --region europe-west1
```

#### Step C: Publish on Gemini Enterprise
```bash
agents-cli publish gemini-enterprise \
  --gemini-enterprise-app-id projects/YOUR_PROJECT_ID/locations/global/collections/default_collection/engines/YOUR_APP_ID \
  --display-name "GKE VPA Rightsizer Agent" \
  --description "Scans GKE clusters for Vertical Pod Autoscaler recommendations and compiles an interactive dashboard."
```
