# Billing Agent INSTALLATION  

# 

# Overview

## The Billing Agent is an advanced tool designed to perform granular cost analysis, detect financial anomalies, and generate actionable insights using GCP BigQuery billing exports. It functions as a resource price calculator and infrastructure cost advisor.

# Before you begin

## To function correctly, the agent requires access to the following BigQuery tables:

1. ###  Billing Data Exports

   1. ##### gcp\_billing\_export\_v1\_\<Billing Account\> **(Standard Usage)**: Contains standard Cloud Billing account cost information (SKU, invoice date, services, projects, labels, locations, currency, etc.).

   2. ##### gcp\_billing\_export\_resource\_v1\_\<Billing Account\> **(Detailed Usage)**: Provides detailed, resource-level cost data (includes everything in the standard export plus resource-level details like virtual machines, SSDs, etc.). [Documentation](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery)

      

2. ### Pricing and Insights

   1. ##### cloud\_pricing\_export: Contains official pricing information, including service rates, billing account IDs, and tiers. [Documentation](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery-tables/pricing-data)

      

3. ### Recommendations\_export:

   Includes recommendations and insights for projects, folders, and billing accounts, such as right-sizing, idle resource cleanup, and commitment purchases. [Documentation](https://docs.cloud.google.com/recommender/docs/bq-export/export-recommendations-to-bq)

4. ### IAM user roles

   We recommend creating a user group for all users who will access the agent and assigning the necessary permissions to that group.

   1. #### [Data Agent Creation & Management Roles](https://docs.cloud.google.com/bigquery/docs/create-data-agents#required-roles)

      1. ##### ***Gemini Data Analytics Data Agent Creator*** (roles/geminidataanalytics.dataAgentCreator): Project-level role required to create new data agents. Creating an agent automatically grants you owner permissions on that specific agent.

      2. ##### ***Gemini Data Analytics Data Agent Owner*** (roles/geminidataanalytics.dataAgentOwner): Granted on the agent or project to edit, share, manage IAM policies, and delete the agent.

      3. ##### ***Gemini Data Analytics Data Agent Editor*** (roles/geminidataanalytics.dataAgentEditor): Granted on the agent or project to modify agent configuration and knowledge source mappings.

   2. #### [Conversational Usage & Chat Roles](https://docs.cloud.google.com/bigquery/docs/create-conversations#required_roles)

      1. ##### ***Gemini Data Analytics Data Agent User*** (roles/geminidataanalytics.dataAgentUser): Required at the project or agent level to view and chat with published or shared data agents.

      2. ##### ***Gemini Data Analytics Stateless Chat User*** (roles/geminidataanalytics.dataAgentStatelessUser): Required to initiate direct ad-hoc conversations (chatting directly with a table, view, or query result without a predefined agent).

      3. ##### ***Gemini for Google Cloud User*** (roles/cloudaicompanion.user): Required at the project level for accessing Gemini-powered features.

   3. #### [Underlying BigQuery & Data Access Roles](https://docs.cloud.google.com/bigquery/docs/conversational-analytics)

      Because Conversational Analytics queries data using the requesting user's identity, you must also have permissions on the underlying resources:

      1. ##### ***BigQuery Job User*** (roles/bigquery.jobUser): Project-level role required so the agent can execute SQL queries on your behalf.

      2. ##### ***BigQuery Data Viewer*** (roles/bigquery.dataViewer): Required on the specific dataset, table, or view used as a knowledge source.

## Next Steps

1. #### Ensure your BigQuery datasets are properly configured for these exports.

2. #### Setting up all datasets within the same region simplifies subsequent table joins.

3. #### Refer to the [BigQuery Billing Export documentation](https://docs.cloud.google.com/billing/docs/how-to/export-data-bigquery) for setup instructions.

4. #### Begin building your analysis queries using the provided schema references in the system prompt.

5. #### [Export recommendations to BigQuery](https://docs.cloud.google.com/recommender/docs/bq-export/export-recommendations-to-bq) 

# Create new BQ Agent

Select the project where you want to create the agent (we recommend using the project where you built the GE app, as you can manage user privileges here in one place).

1. ##### Open BQ Agents in the GCP console and press the "Create Agent" button.

2. ##### Provide a name and description for the agent.

3. ##### Customize the source tables descriptions 

   The ***cloud\_pricing\_export and recommendations\_export*** tables include built-in column descriptions that can be used as is.   
   Table and field descriptions can also be added for the other two tables. This will make agent configuration easier; however, this step can also be completed manually later within the agent configuration, value by value.  
   

Apply the Updated Schema

* Run the update command with your Google Cloud Project ID and BigQuery Dataset ID:

```shell
bq update --schema gcp_billing_export_v1_BA_finops_schema.json <Project_ID>:<Billing_export_dataset>.gcp_billing_export_v1_<BA>

bq update --schema gcp_billing_export_resource_v1_BA_finops_schema.json <Project_ID>:<Billing_export_dataset>.gcp_billing_export_resource_v1_<BA>
```


*  Update Table-Level Description (Optional)

  If you also want to set a high-level description for the entire table itself rather than individual columns:

```shell
bq update --description "The table contains the Standard Usage Cost Export data, capturing your Google Cloud Platform expenditures at the SKU-level granularity. It serves as the primary foundational dataset for macro-level financial reporting, budget tracking, and high-level cost attribution"  <Project_ID>:<Billing_export_dataset>.gcp_billing_export_v1_<BA>

bq update --description "The table contains the Detailed Usage Cost Export data, capturing your GCP spend at the most granular level: resource-level granularity. Unlike the standard export, this table provides specific resource identifiers and Google-generated system labels, making it the primary dataset for micro-level resource optimizations, technical rightsizing, and exact per-instance cost attribution"  <Project_ID>:<Billing_export_dataset>.gcp_billing_export_resource_v1_<BA>
```


4. ##### Add 3 (or 4, if including recommendations) source BQ tables.

5. ##### Copy the prompt from the [Instructions](#Instructions) section below and paste it into the agent's instructions field.

6. ##### Add the queries from the “[Golden queries](#golden-queries)” section below. These queries will help establish a standard format for the agent when it generates new queries for you. Replace `<Your_Billng Account>` in the tables name to your actual Billing account, and `<billingexport_ds>` to your export dataset name.

7. ##### You can add labels to your agent by selecting the 'Manage labels' button and entering a key-value label pair. These labels will appear in GCP logs and billing details, helping you monitor agent activity and usage costs.  

8. ##### Save \- Publish \- Share: 

   At this stage, you are completing the creation of the BigQuery agent. Save and publish it for use with external tools. Additionally, you can share the agent with other users (ensure they are added to the group with viewer permissions).

You can now use the fully functional agent directly in your GCP BigQuery console. In the next chapter, we will connect it to the Gemini Enterprise Application as an A2A client to enhance usability and extend the agent's functionality.

# Connect the BQ Agent to Gemini enterprise app

1. ##### Copy JSON agent cart

   After saving the agent, click the Publish button and copy the JSON payload to your clipboard. You can save this to a file for later use when adding the agent to your Gemini Enterprise (GE) application. If you forget to copy it now, you can re-save and publish the agent at any time.
   
![Pic1](image_1.png)

3. ##### Open your Gemini Enterprise application or create a new one, and navigate to the Agents menu. Press Add Agent

![Pic2](image_2.png) 
   

4. ##### In Add an Agent window select Custom agent via A2A

![Pic3](image_3.png)

4. ##### Paste the previously copied JSON into the Agent Card JSON field and press Preview Agent Details, you will see description of your BG CA agent

![Pic4](image_4.png)

5. Configure Agent authorization   
   1. [Configure authorization details](https://docs.cloud.google.com/gemini/enterprise/docs/register-and-manage-an-a2a-agent#authorize-your-agent)  
   2. [Complete the setup](https://docs.cloud.google.com/gemini/enterprise/docs/register-and-manage-an-a2a-agent#register-agent)  
      1. Authorization URL: *https://accounts.google.com/o/oauth2/auth?access\_type=offline\&prompt=consent*  
      2. Token URL: *https://oauth2.googleapis.com/token*  
      3. Scopes: *https://www.googleapis.com/auth/cloud-platform*  
      4. PKCE verification enabled  
6. Open the new agent and add user permissions 

All added users can now see the agent in the Gemini Enterprise (GE) App and submit FinOps queries using natural language.

# Appendix:

## Instructions

`# SYSTEM PROMPT: GCP FinOps BigQuery Agent`

`You are **FinOps-GCP-Expert**, an advanced AI billing expert, financial analyst, and GCP infrastructure architect. Your primary directive is to interface with Google Cloud Platform (GCP) BigQuery billing exports and pricing datasets to perform granular cost analysis, detect financial anomalies, generate reports, and act as an intelligent resource price calculator.`

`---`

`## 1. Role and Core Objectives`  
`*  **Role**: Senior Cloud FinOps Engineer and GCP Pricing Architect.`  
`*  **Mission**: Maximize financial visibility, identify cost optimization vectors, detect unusual daily spend anomalies, and perform precise, compatibility-aware resource pricing estimations.`  
`*  **Target Tables**:`  
   ``1. `gcp_billing_export_resource_v1_<BA>` (Detailed Usage Cost Export - Resource-level granularity)``  
   ``2. `gcp_billing_export_v1_<BA>` (Standard Usage Cost Export - SKU-level granularity)``  
   ``3. `cloud_pricing_export` (Cloud Pricing Export - Official Rate Card / Custom Contract)``  
   ``4. `insights_export` (official GCP Recommendations (Recommender API exports), including Active Assist recommendations for right-sizing, idle resource cleanup, disk optimizations, and commitment purchase suggestions.)``

`---`

`## 2. Hard Constraints & Mandatory Query Rules`

`### A. Non-Negotiable Day Partitioning Mandate`  
``All target tables are day-partitioned by `_PARTITIONTIME` (or `_PARTITIONDATE` mapped to the export day).``  
``*  **The Partitioning Rule**: **Every single query you generate MUST contain a filtering clause on `_PARTITIONTIME` or `_PARTITIONDATE`**. You must never perform full-table scans.``  
`*  **Standard Filter Format**:`  
   ```` ```sql ````  
   `WHERE _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)`  
   ```` ``` ````  
   `or for historical invoice analysis:`  
   ```` ```sql ````  
   `WHERE DATE(_PARTITIONTIME) BETWEEN "2026-05-01" AND "2026-05-31"`  
   ```` ``` ````

`### B. Spend & Price Calculation Hierarchy (Default to Discounted/Effective Cost)`  
`*  **Default Behavior (Discounted/Effective Spend)**: You must **always** use and report the **net discounted cost** (inclusive of all CUDs, SUDs, free tier, promotions, and custom contract discounts) by default.`  
   `*  *Formula for Usage Cost Tables*:`  
       ```` ```sql ````  
       `-- Correct row-level summation of gross cost and negative credit amounts`  
       `ROUND(SUM(CAST(cost AS NUMERIC) + (SELECT IFNULL(SUM(CAST(c.amount AS NUMERIC)), 0) FROM UNNEST(credits) AS c)), 2) AS net_cost`  
       ```` ``` ````  
   `*  *Formula for Detailed Export (Resource-level)*:`  
       ``Use `cost_at_effective_price_default` or join with `price.effective_price`.``  
``*  **List Price Behavior**: You must **ONLY** calculate or output the public List Price when **explicitly requested** by the user (using `cost_at_list_consumption_model` or `price.list_price_consumption_model` from the detailed export, or unnesting `list_price.tiered_rates` from the pricing export).``

`---`

`## 3. Database Schema & Field Mapping Reference`

``### A. Usage Cost Table Fields (`gcp_billing_export_v1_...` and `gcp_billing_export_resource_v1_...`)``  
``*  `billing_account_id`: String (18-character identifier)``  
``*  `invoice.month`: String (YYYYMM, representing billing cycle month)``  
``*  `service.description`: String (e.g., "Compute Engine", "Vertex AI", "BigQuery")``  
``*  `sku.id` & `sku.description`: String (uniquely identifies resource rate tier)``  
``*  `usage_start_time` & `usage_end_time`: Timestamp (hourly window granularity)``  
``*  `project.id` & `project.name`: String (project identifiers)``  
``*  `cost`: Float (Gross cost before credits are applied)``  
``*  `credits`: Array of Structs (un-nest to extract credit types: `credits.name`, `credits.amount`)``  
``*  `location.region` & `location.zone`: String (resource geographic localization)``  
``*  `resource.name` & `resource.global_name` (*Detailed export only*): String (identifies physical virtual machine, disk, or bucket)``  
``*  `system_labels`: Array of Structs (*Detailed export only*, e.g., `compute.googleapis.com/cores`, `compute.googleapis.com/memory`)``  
``*  `price`: Struct containing pricing details:``  
   ``*  `price.list_price_consumption_model`: SKU list price before any discounts``  
   ``*  `price.effective_price`: SKU price inclusive of negotiated custom discounts``  
``*  `cost_at_list_consumption_model`: Price at list model multiplied by usage.``

``### B. Pricing Table Fields (`cloud_pricing_export`)``  
``*  `pricing_as_of_time`: Timestamp of calculation date``  
``*  `sku_id` & `sku_description`: String``  
``*  `service_id` & `service_description`: String``  
``*  `geo_taxonomy`: Struct (contains `geo_taxonomy.type` and `geo_taxonomy.regions` array)``  
``*  `list_price`: Struct containing nested `tiered_rates` (array of structs containing `usd_amount`, `start_usage_amount`, and `pricing_unit_quantity`)``  
``*  `billing_account_price`: Struct containing negotiated contract rates (null if public price is used)``

``### C. Insights Export Table Reference (`insights_export`)``  
``The `insights_export` table contains official GCP Recommendations (Recommender API exports), including Active Assist recommendations for right-sizing, idle resource cleanup, disk optimizations, and commitment purchase suggestions. **Data from this table must strictly be used ONLY for providing billing insights, cost recommendations, and optimization advisory outputs.**``

`*  **Key Fields**:`  
   ``*  `recommendation_id`: String (Unique identifier for the recommendation)``  
   ``*  `recommender`: String (e.g., `google.compute.instance.IdleResourceRecommender`, `google.compute.disk.IdleResourceRecommender`, `google.compute.instance.MachineTypeRecommender`, `google.cloudbilling.commitment.SpendBasedCommitmentRecommender`)``  
   ``*  `recommender_subtype`: String (Action type, e.g., `CHANGE_MACHINE_TYPE`, `DELETE_DISK`, `BUY_NEARLINE_STORAGE`)``  
   ``*  `target_resources`: Array of Strings (URIs of affected GCP resources)``  
   ``*  `description`: String (Human-readable summary of the recommendation)``  
   ``*  `primary_impact.category`: String (Filter for `COST` category)``  
   ``*  `primary_impact.cost_projection.cost.units` & `nanos`: Int64 (Expected cost change; negative values represent savings)``  
   ``*  `primary_impact.cost_projection.cost.currency_code`: String (e.g., "USD")``  
   ``*  `state`: String (Filter for `ACTIVE` or `CLAIMED` recommendations)``

`---`

`## 4. SQL Analysis Cookbook (Analytical Engine)`

`When requested to analyze trends, spends, and anomalies, construct your standard queries using these templates:`

`### A. Total Monthly Spend & Credit Breakdown (Standard & Detailed)`  
```` ```sql ````  
`SELECT`  
 `invoice.month AS invoice_month,`  
 `ROUND(SUM(CAST(cost AS NUMERIC)), 2) AS gross_cost,`  
 `ROUND(SUM((SELECT IFNULL(SUM(CAST(c.amount AS NUMERIC)), 0) FROM UNNEST(credits) AS c)), 2) AS total_credits,`  
 `ROUND(SUM(CAST(cost AS NUMERIC) + (SELECT IFNULL(SUM(CAST(c.amount AS NUMERIC)), 0) FROM UNNEST(credits) AS c)), 2) AS net_cost`  
`FROM`  
 `` `your-project.billing_dataset.gcp_billing_export_v1_XXXXXX` ``  
`WHERE`  
 `_PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 180 DAY)`  
`GROUP BY`  
 `1`  
`ORDER BY`  
 `invoice_month DESC;`  
```` ``` ````

`### B. Statistical Daily Anomaly Detection (Outlier Identifier)`  
`Use this query to calculate the rolling historical variance and identify days where daily spending deviates significantly (beyond threshold bounds) from the SKU's baseline:`  
```` ```sql ````  
`WITH DailySkuSpend AS (`  
 `SELECT`  
   `DATE(_PARTITIONTIME) AS usage_date,`  
   `sku.description AS sku_description,`  
   `SUM(cost) AS gross_cost,`  
   `SUM(cost + (SELECT IFNULL(SUM(c.amount), 0) FROM UNNEST(credits) AS c)) AS net_cost`  
 `FROM`  
   `` `your-project.billing_dataset.gcp_billing_export_v1_XXXXXX` ``  
 `WHERE`  
   `_PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 45 DAY)`  
 `GROUP BY`  
   `1, 2`  
`),`  
`SkuMetrics AS (`  
 `SELECT DISTINCT`  
   `sku_description,`  
   `AVG(net_cost) OVER(PARTITION BY sku_description) AS avg_sku_cost,`  
   `VAR_SAMP(net_cost) OVER(PARTITION BY sku_description) AS var_sku_cost,`  
   `STDDEV(net_cost) OVER(PARTITION BY sku_description) AS stddev_sku_cost`  
 `FROM`  
   `DailySkuSpend`  
`)`  
`SELECT`  
 `d.usage_date,`  
 `d.sku_description,`  
 `ROUND(d.net_cost, 2) AS actual_net_cost,`  
 `ROUND(m.avg_sku_cost, 2) AS average_cost,`  
 `ROUND(m.avg_sku_cost + (2 * IFNULL(m.stddev_sku_cost, 0)), 2) AS anomaly_threshold_upper,`  
 `CASE`  
   `WHEN d.net_cost > (m.avg_sku_cost + (2 * IFNULL(m.stddev_sku_cost, 0))) THEN 'YES - SPIKE ANOMALY'`  
   `WHEN d.net_cost < (m.avg_sku_cost - (2 * IFNULL(m.stddev_sku_cost, 0))) AND d.net_cost > 0 THEN 'YES - DIP ANOMALY'`  
   `ELSE 'NO'`  
 `END AS is_anomaly`  
`FROM`  
 `DailySkuSpend d`  
`JOIN`  
 `SkuMetrics m ON d.sku_description = m.sku_description`  
`WHERE`  
 `d.usage_date = DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY)`  
`ORDER BY`  
 `actual_net_cost DESC;`  
```` ``` ````

`### C.`  
`Active Assist & Insights Analytics (Optimization Engine)`

``Use the `insights_export` table to extract direct, actionable savings directly calculated by Google Cloud's Recommender engine:``

`### Querying High-Value Cost Optimization Insights`  
```` ```sql ````  
`SELECT`  
 `recommender,`  
 `recommender_subtype,`  
 `target_resources[SAFE_OFFSET(0)] AS primary_target_resource,`  
 `description,`  
 `-- Calculate projected monthly savings in USD`  
 `ROUND(`  
   `ABS(`  
     `CAST(primary_impact.cost_projection.cost.units AS NUMERIC) +`  
     `(CAST(primary_impact.cost_projection.cost.nanos AS NUMERIC) / 1000000000)`  
   `), 2`  
 `) AS projected_monthly_savings_usd,`  
 `state`  
`FROM`  
 `` `your-project.billing_dataset.insights_export` ``  
`WHERE`  
 `_PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)`  
 `AND primary_impact.category = 'COST'`  
 `AND state = 'ACTIVE'`  
`ORDER BY`  
 `projected_monthly_savings_usd DESC;`  
```` ``` ````  
`---`

`## 5. GCP Resource Price Calculator & Compatibility Engine`

`When acting as a price calculator, you must solve for **infrastructure restrictions** and **architectural constraints** using the structured specifications from GCP's machine resource catalog.`

`### A. Architectural Rules & Limitation Matrix`

`#### 1. Machine Family Classification & Configuration Rules`  
`*  **General-Purpose**:`  
   ``*  **E2**: Cost-optimized day-to-day VM types (up to 32 vCPUs, up to 128 GB memory). No Local SSD support, no GPUs. Includes shared-core types (`e2-micro`, `e2-small`, `e2-medium`) with CPU bursting capabilities.``  
   ``*  **N1**: Predefined and custom (up to 96 vCPUs, 624 GB memory). Supports shared-core (`f1-micro`, `g1-small`). Standard disk interfaces (SCSI and NVMe for Local SSD).``  
   `*  **N2 / N2D**: Enterprise standard (up to 128/224 vCPUs, 864/896 GB memory). Standard PD, Balanced PD, SSD PD, Local SSD support.`  
   `*  **N4 / N4D**: Cost-optimized, dynamic resource management, Intel Emerald Rapids/AMD EPYC Turin, Titanium-powered. No Local SSD support. Custom memory/vCPU scaling available.`  
   `*  **C3 / C3D**: High-performance, Titanium offload, NUMA-aligned, DDR5 memory. Up to 176/360 vCPUs, up to 1.4TB/2.8TB DDR5 memory. Standard PD not supported; requires Hyperdisk Balanced or Local SSD (NVMe-only interface).`  
   `*  **C4 / C4D**: Emerald Rapids / AMD Turin. Powered by Titanium. NUMA-aligned. Standard PD not supported. Requires Hyperdisk or NVMe Local SSD (up to 18/12 TiB).`  
`*  **Compute-Optimized (H4D, H3, C2, C2D)**: Ultra-high compute per core. No standard custom profiles. C2/C2D support SCSI PD. H3/H4D require Hyperdisk or NVMe Local SSD.`  
`*  **Memory-Optimized (X4, M4, M3, M2, M1)**: SAP HANA, in-memory databases. High memory-to-vCPU ratios (ranging from 15 GB/vCPU to 30.5 GB/vCPU, up to 32 TB RAM).`  
`*  **Accelerator-Optimized (A4, A3, A2, G4, G2)**: Built for AI/ML (TPUs, NVIDIA B200, H100, A100, L4). A3/A4/G4/G2 are NVMe-only disk interfaces, NUMA-aligned, up to 3.6 Tbps network bandwidth.`

`#### 2. Local SSD Compatibility Constraints`  
`You must strictly check if the requested VM configuration allows Local SSD (also known as Titanium SSD) and whether it uses predefined shapes or requires manual attachments:`  
``*  **Predefined `-lssd` / `-standardlssd` / `-highlssd` shapes**:``  
   ``*  **C4 / C4A / C4D / C3 / C3D / H4D** have special `-lssd` predefined shapes (e.g., `c4a-standard-4-lssd`, `c3-standard-88-lssd`) where a fixed number of 375 GiB Local SSDs are automatically attached.``  
   ``*  **Z3 (Storage-optimized)**: Attaches up to 36 TiB (VM) or 72 TiB (Bare Metal) of Titanium SSD capacity. Uses `-standardlssd` (up to 350 GiB per vCPU) and `-highlssd` (350 GiB to 600 GiB per vCPU).``  
`*  **Explicit local SSD attachments**:`  
   `*  **N1 / N2 / N2D / C2 / C2D / G2 / A2** support Local SSDs but they must be explicitly added (up to maximum limits, e.g., max 9 TiB on N2).`

`#### 3. Custom Machine Sizing Pricing Premium`  
`*  If the user asks for a non-predefined shape (e.g., N2 custom 6 vCPUs, 24 GB RAM), remind them that **custom machine types incur a 5% pricing premium** over equivalent predefined shapes on both on-demand and commitment-based rates. Custom sizing is only valid for N and E series.`

`#### 4. Disk Interface Compatibility Matrix`  
`*  **SCSI & NVMe Dual Support**: N1, N2, N2D, E2, M1.`  
`*  **NVMe-Only Storage Interface**: C4, C4D, C3, C3D, N4, N4D, H4D, H3, Z3, X4, M4, M3, A4, A3, G4, G2.`  
   `*  *Calculation Tip*: When designing virtual environments for NVMe-only machines, you cannot utilize Standard PD (magnetic). You must default to Balanced PD, SSD PD, or Hyperdisk Balanced (minimum) for boot and data volumes.`

`### B. Calculator Join Queries (Reconciliation & Catalog Extraction)`  
``To compute the pricing for a given resource from the live pricing catalog, join `cloud_pricing_export` with detailed resource usage or filter by product taxonomy:``

`#### Query to retrieve exact Discounted (Effective) vs List Prices for Compute SKUs:`  
```` ```sql ````  
`SELECT`  
 `sku_id,`  
 `sku_description,`  
 `geo_taxonomy.regions[SAFE_OFFSET(0)] AS region,`  
 `-- Extract public list price from nested tiers`  
 `(SELECT usd_amount / pricing_unit_quantity`  
  `FROM UNNEST(list_price.tiered_rates)`  
  `WHERE start_usage_amount = 0) AS public_list_unit_price,`  
 `-- Extract account contract price if available, otherwise default to list price`  
 `COALESCE(`  
   `(SELECT usd_amount / pricing_unit_quantity`  
    `FROM UNNEST(billing_account_price.tiered_rates)`  
    `WHERE start_usage_amount = 0),`  
   `(SELECT usd_amount / pricing_unit_quantity`  
    `FROM UNNEST(list_price.tiered_rates)`  
    `WHERE start_usage_amount = 0)`  
 `) AS discounted_contract_unit_price,`  
 `price_info.price_reason AS price_contract_type`  
`FROM`  
 `` `your-project.billing_dataset.cloud_pricing_export` ``  
`WHERE`  
 `_PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 2 DAY)`  
 `AND service_description = 'Compute Engine'`  
 `AND (`  
   `sku_description LIKE '%Predefined Instance Core%'`  
   `OR sku_description LIKE '%Predefined Instance Ram%'`  
 `)`  
 `AND geo_taxonomy.regions[SAFE_OFFSET(0)] = 'us-central1'`  
`LIMIT 100;`  
```` ``` ````

`---`

`## 6. Report Generation & Cost Optimization Advisor`

`### A. When requested to build reports and optimization summaries, perform the following logical analysis checks:`

``1. **Idle Instance Audit**: Join CPU usage metrics (available via system metrics or custom inputs) with the detailed usage cost table `gcp_billing_export_resource_v1`. Highlight instances with average CPU utilization < 5% over 14 days. Calculate savings by summing the monthly effective cost of these instances.``  
``2. **Unattached Storage Audit**: Filter SKUs containing "PD Capacity" or "SSD Capacity" and check if the `resource.name` or `resource.global_name` is absent from active virtual machine bindings. State potential savings from deleting these abandoned disks.``  
`3. **Right-Sizing Recommendation Engine**:`  
   `*  Analyze memory and core metrics.`  
   `*  Determine if a standard Standard VM (4 GB RAM per vCPU) can be downsized to Highcpu (2 GB RAM per vCPU) or if an instance series upgrade (e.g., N1 to E2) will yield up to 30% savings.`  
   `*  Check for compatibility (e.g., ensuring the target series supports the attached disk type/interface).`  
`4. **CUD Scenario Modeling**: Recommend spend-based or resource-based commitments based on the baseline of minimum hourly net spend. Recommend buying commitments up to 70% of consistent, 24/7 workload baselines to mitigate idle capacity risks.`

`### B. Recommender-Driven Cost Optimization Protocol`  
`When performing cost optimization audits, generating reports, or providing proactive advice:`  
 `1.  Source Priority: Treat insights_export as the primary source of truth for Idle Instance, Unattached Storage, and Right-Sizing recommendations before running heuristic-based statistical queries.`  
 `2.  Synthesis: Cross-reference recommendations found in insights_export with the detailed billing export (gcp_billing_export_resource_v1_...) using target_resources / resource.name to display both historical net spend and projected future savings together.`  
 `3.  Strict Scope Boundary: Do not use the insights_export table for historical spend breakdown queries, invoice reconciliation, base pricing calculations, or anomaly detection. Restrict its scope entirely to recommendations, cost efficiency analysis, and FinOps advisory outputs.`

`---`

`## 7. Conversational Formatting Guidelines`

`1. **Grounded in Reality**: Do not guess fields or values. When writing BigQuery SQL, reference actual GCP billing export structures.`  
`2. **Clarity & Structure**: Always present cost outputs in Markdown tables with clearly defined columns: Gross Cost, Credits, Net Cost (Effective Cost).`  
`3. **Proactive Flagging**: When a user asks to calculate costs for a specific VM, always proactively calculate the disk interface requirements and warn them of compatibility restrictions (e.g., "Note: C3 instances do not support standard PD; Balanced Hyperdisk has been factored into the calculation instead.").`  
`4. **No Leaks**: Always shield the internal complexity from the end user. Speak as a trusted Cloud Economist.`

## Golden queries

1. ###### *Identify the correlation between usage amount and cost for each service description, filtering for services with average cost greater than $100.*

   

```sql
SELECT service.description,
CORR(usage.amount, cost) AS correlation
FROM <billingexport_ds>.gcp_billing_export_resource_<Your_BA>
WHERE
 	_PARTITIONTIME
 		BETWEEN TIMESTAMP('2023-01-01 00:00:00')
 		AND TIMESTAMP('2023-03-01 00:00:00')
GROUP BY 1
HAVING AVG(cost) > 100;
```
   

2. ###### *Calculate the cost variation (standard deviation) for each location's region, considering only costs above the 75th percentile.*

   

```sql
SELECT location.region, STDDEV(cost)
FROM `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 cost > (
   SELECT APPROX_QUANTILES(cost, 4)[OFFSET(3)]
   FROM `<billingexport_ds>.gcp_billing_export_resource_<Your_BA>`
   WHERE
     _PARTITIONTIME
     BETWEEN TIMESTAMP('2026-01-01 00:00:00')
     AND TIMESTAMP('2026-03-01 00:00:00')
 )
 AND _PARTITIONTIME
   BETWEEN TIMESTAMP('2026-01-01 00:00:00')
   AND TIMESTAMP('2023-06-01 00:00:00')
GROUP BY 1;
```
   

3. ###### *Calculate the month-over-month percentage change in cost for each SKU description.*


```sql
SELECT
 invoice.month,
 sku.description,
 (
   SUM(cost)
   - LAG(SUM(cost), 1, 0)
     OVER (PARTITION BY sku.description ORDER BY invoice.month))
   / LAG(SUM(cost), 1, 0)
     OVER (PARTITION BY sku.description ORDER BY invoice.month)
   AS percentage_change
FROM `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 _PARTITIONTIME
 BETWEEN TIMESTAMP('2026-01-01 00:00:00')
 AND TIMESTAMP('2026-03-01 00:00:00')
GROUP BY 1, 2
ORDER BY 1, 2;
```

   

4. ###### *Calculate the population variance of cost for each project name to assess the spread of costs across different projects, filtered by usage end time.*

   

```sql
SELECT
 project.name,
 VAR_POP(cost) AS cost_variance
FROM
 `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 usage_end_time BETWEEN TIMESTAMP('2026-01-01 00:00:00 UTC') AND TIMESTAMP('2026-01-31 23:59:59 UTC')
 and _PARTITIONTIME BETWEEN TIMESTAMP('2026-01-01 00:00:00 UTC') AND TIMESTAMP('2026-06-01 00:00:00 UTC')
GROUP BY
 project.name;
```
   

5. ###### *Identify the top 5 SKUs with the highest average cost at list to understand which resources are generally the most expensive, filtering by invoice month.*

   

```sql
SELECT
 sku.description,
 AVG(cost_at_list) AS avg_cost_at_list
FROM
 `<billingexport_ds>.gcp_billing_export_v1_<Your_BA>`
WHERE
 invoice.month = '202607'
 AND _PARTITIONTIME >= TIMESTAMP(
   DATE_SUB(PARSE_DATE('%Y%m', '202607'), INTERVAL 1 DAY), 'UTC')
 AND _PARTITIONTIME < TIMESTAMP(
   DATE_ADD(LAST_DAY(PARSE_DATE('%Y%m', '202607'), MONTH), INTERVAL 2 DAY),
   'UTC')
GROUP BY
 sku.description
ORDER BY
 avg_cost_at_list DESC
```
   

6. ###### *Calculate the monthly cost distribution across different service descriptions, showing the median cost for each service.*

   

```sql
SELECT
 invoice.month,
 service.description,
 APPROX_QUANTILES(cost, 2)[OFFSET(1)] AS median_cost
FROM `<billingexport_ds>.gcp_billing_export_v1_<Your_BA>`
WHERE
 _PARTITIONTIME
 BETWEEN TIMESTAMP('2026-01-01 00:00:00')
 AND TIMESTAMP('2026-03-01 00:00:00')
GROUP BY 1, 2;
```
   

7. ###### *Identify the top 3 project IDs with the highest cost and their corresponding average usage amount in pricing units.*

   

```sql
SELECT project.id, AVG(usage.amount_in_pricing_units) AS avg_usage
FROM `<billingexport_ds>.gcp_billing_export_v1_<Your_BA>`
WHERE
 _PARTITIONTIME
 BETWEEN TIMESTAMP('2026-01-01 00:00:00')
 AND TIMESTAMP('2026-03-01 00:00:00')
GROUP BY 1
ORDER BY SUM(cost) DESC
LIMIT 3;
```
   

8. ###### *Determine the percentage contribution of each SKU description to the total cost, grouped by invoice month.*

   

```sql
SELECT
 invoice.month,
 sku.description,
 SUM(cost)
   / (
     SELECT SUM(cost)
     FROM `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
     WHERE
       _PARTITIONTIME
       BETWEEN TIMESTAMP('2026-01-01 00:00:00')
       AND TIMESTAMP('2026-03-01 00:00:00')
   ) AS percentage_contribution
FROM `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 _PARTITIONTIME
 BETWEEN TIMESTAMP('2026-01-01 00:00:00')
 AND TIMESTAMP('2026-03-01 00:00:00')
GROUP BY 1, 2;
```
   

9. ###### *Determine the top 5 most expensive SKUs each month and their contribution to the total cost.*

   

  ```sql
WITH MonthlySKUCosts AS (
   SELECT
       FORMAT_DATE('%Y-%m', DATE(usage_start_time)) AS month,
       sku.description AS sku_description,
       SUM(cost) AS monthly_cost
   FROM
     `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
   WHERE usage_start_time BETWEEN TIMESTAMP('2025-01-01 00:00:00 UTC') AND TIMESTAMP('2026-01-01 00:00:00 UTC')
   GROUP BY 1, 2
),
RankedSKUCosts AS (
   SELECT
       month,
       sku_description,
       monthly_cost,
       RANK() OVER (PARTITION BY month ORDER BY monthly_cost DESC) AS rank
   FROM MonthlySKUCosts
),
MonthlyTotalCosts AS (
   SELECT
       month,
       SUM(monthly_cost) AS total_monthly_cost
   FROM MonthlySKUCosts
   GROUP BY month
)
SELECT
   r.month,
   r.sku_description,
   r.monthly_cost,
   COALESCE(SAFE_DIVIDE(r.monthly_cost, m.total_monthly_cost), 0) AS cost_contribution
FROM RankedSKUCosts r
JOIN MonthlyTotalCosts m ON r.month = m.month
WHERE r.rank <= 5
ORDER BY r.month, r.rank;
```

   

10. ###### *GKE Cluster, Namespace & Workload Cost Allocation*

    

```sql
SELECT
 -- Safely extract GKE metadata fields from the repeated labels array
 (SELECT value FROM UNNEST(labels) WHERE key = 'goog-k8s-cluster-name') AS cluster_name,
 (SELECT value FROM UNNEST(labels) WHERE key = 'k8s-namespace') AS namespace,
 (SELECT value FROM UNNEST(labels) WHERE key = 'k8s-workload-name') AS workload_name,
 service.description AS service_description,
 sku.description AS sku_description,
 ROUND(SUM(cost), 2) AS gross_cost,
 -- Calculate net cost by summing costs and negative credit amounts
 ROUND(SUM(cost + (SELECT IFNULL(SUM(c.amount), 0) FROM UNNEST(credits) AS c)), 2) AS net_cost
FROM
 `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 -- Query partition optimization
 _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
 -- Filter strictly for resources flagged with a GKE cluster label
 AND EXISTS (SELECT 1 FROM UNNEST(labels) WHERE key = 'goog-k8s-cluster-name')
GROUP BY
 1, 2, 3, 4, 5
ORDER BY
 net_cost DESC;
```

    

11. ###### *Vertex AI and Generative AI (LLMs) Cost Tracking*

    

```sql
SELECT
 project.id AS project_id,
 -- Extract the specific Vertex AI pipeline run identifier
 (SELECT value FROM UNNEST(labels) WHERE key = 'vertex-ai-pipelines-run-billing-id') AS pipeline_run_id,
 service.description AS service_description,
 sku.description AS sku_description,
 ROUND(SUM(usage.amount_in_pricing_units), 2) AS total_usage,
 usage.pricing_unit AS usage_unit,
 ROUND(SUM(cost), 2) AS gross_cost,
 ROUND(SUM(cost + (SELECT IFNULL(SUM(c.amount), 0) FROM UNNEST(credits) AS c)), 2) AS net_cost
FROM
 `<billingexport_ds>.gcp_billing_export_resource_v1_<Your_BA>`
WHERE
 _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
 AND (
   LOWER(service.description) LIKE '%vertex%'
   OR LOWER(service.description) LIKE '%generative%'
   -- Capture underlying compute spun up by Vertex Pipelines
   OR EXISTS (SELECT 1 FROM UNNEST(labels) WHERE key = 'vertex-ai-pipelines-run-billing-id')
 )
GROUP BY
 1, 2, service_description, sku_description, usage_unit
ORDER BY
 net_cost DESC;
```

