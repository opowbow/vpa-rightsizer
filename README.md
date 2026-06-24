# 📊 GKE VPA Rightsizer Agent

An intelligent, multi-agent automated pipeline built with the **Google Agent Development Kit (ADK)**. It automates GKE cluster resource analysis, queries Vertical Pod Autoscaler (VPA) metrics, generates **newly optimized Kubernetes deployment manifests with updated CPU and memory requests**, compiles an interactive web dashboard, and deploys it to Google Cloud Run.

---

## 🚀 Key Features

* **Multi-Agent Pipeline Orchestration**: Implements sequential checks and verification between dedicated subagents to prevent failures and handle automated workflows.
* **Intelligent GKE Scraping (`gke_scraper`)**:
  * Scans multiple GKE clusters dynamically across Google Cloud project(s).
  * Extracts active Vertical Pod Autoscaler (VPA) targets, lower bounds, and upper bounds.
  * Dynamically queries **Cloud Monitoring fallback metrics** for workloads missing VPA definitions.
  * Prompts for missing Project IDs on the fly using the built-in `request_input` tool.
* **Optimized Manifest Generation**: Automatically outputs **new deployment manifest files** with updated container `resources.requests.cpu` and `resources.requests.memory` values applied directly based on recommendations.
* **Dataset & Asset Builder (`web_builder`)**: Parses raw scraper findings into structured JSON and copies updated, clean deployment manifests.
* **Cloud Run Deployment (`cloud_run_deployer`)**: Deploys the Express-based interactive reporting dashboard to Google Cloud Run.


---

## 📝 Auto-Generated Manifest Right-Sizing Example

To ensure GKE workloads are correctly sized without manual intervention, the pipeline automatically writes **updated, clean deployment manifests** for each analyzed workload. 

When a VPA recommendation or Cloud Monitoring metric is resolved, the agent:
1. Strips out read-only cluster status, uid, system annotations, and managed fields.
2. Directly overrides the container's `resources.requests.cpu` and `resources.requests.memory` fields in the template spec with the optimized recommendations.

Below is an example of how a deployment manifest (e.g., `emailservice.yaml`) is modified during the pipeline run:

```diff
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: emailservice
   namespace: default
 spec:
   replicas: 1
   template:
     spec:
       containers:
       - name: server
         image: gcr.io/google-samples/microservices-demo/emailservice:v0.3.5
         resources:
           requests:
-            cpu: 100m
-            memory: 64Mi
+            cpu: 80m      # Automatically updated to VPA target recommendation
+            memory: 120Mi # Automatically updated to VPA target recommendation
```

These ready-to-apply files are saved locally under the git-ignored `results/vpa-<project_id>/vpa-<cluster_name>/<namespace>/` directory and served interactively via the Web Dashboard.

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
