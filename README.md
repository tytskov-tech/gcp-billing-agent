# gcp-billing-agent

## Bridging the Gap Between Engineering and Finance: Talking directly to your AI-Driven GCP Billing 

Cloud financial management (FinOps) has a notorious communication gap.
Engineers live in the Google Cloud Console, developing and tracking architecture changes and infrastructure optimization metrics. Finance, revenue, and procurement teams live in spreadsheets, trying to map raw line-item costs back to business units. When an unexpected spike in cloud spend happens, they email Engineering. Engineering digs through the GCP console, writes a few complex BigQuery SQL queries against the billing export, cross-references it with pricing tables, and eventually realizes a developer left a cluster of GPUs running after a weekend experiment.
The process takes hours, sometimes days. By the time the anomaly is found, the money is spent.
The detection of cloud spend anomalies represents an important link between FinOps and SecOps. While sudden cost fluctuations often highlight inefficiencies or misconfigurations, they are equally indicative of security incidents like resource hijacking or data theft.
The resulting investigation usually involves a high volume of emails, ad-hoc dashboard creation, and delayed answers.
What if your finance, procurement and security teams could simply ask your data for the answers?
"Why did our BigQuery costs spike by 20% last Tuesday?" "What are our top three infrastructure cost-saving recommendations this month, who owns those projects and who deleted a machine last week?"

The core challenge of FinOps isn’t a lack of data — it’s data accessibility. Cloud billing, pricing APIs, sizing recommendations, and audit logs generate massive amounts of telemetry. But the teams who actually need to govern this data — Finance, Procurement, and Security — rarely know how to query it.
To bridge this gap, we built a BigQuery Conversational Agent powered by Gemini Enterprise. 

GCP provides excellent tools to capture this data. You can export Cloud Billing to BigQuery, ingest Cloud Audit Logs, and pull sizing suggestions from the Recommender API.
However, raw BigQuery tables are useless to a financial analyst who doesn't write SQL. Dashboards (like Looker or Grafana) help, but they are rigid. The moment an analyst has a question that falls outside the dashboard’s predefined filters, they are back to filing a Jira ticket for the data engineering team.

## The Solution: A Conversational FinOps Agent
A BigQuery Conversational Analytics Agent is an AI-powered bridge between human language and complex cloud data. Instead of requiring a user to know database schemas or SQL syntax, the agent acts as a bilingual translator. It takes a question asked in natural language, uses a Large Language Model to translate that intent into optimized BigQuery SQL, executes the query against the data warehouse, and then translates the resulting rows and columns back into a natural, conversational response. It effectively turns a massive, rigid data warehouse into an interactive chatbot.
To democratize cloud cost visibility, we connected an LLM directly to our FinOps data warehouse.
[For more information and examples, please visit ](https://medium.com/google-cloud/cloud-financial-management-finops-has-a-notorious-communication-gap-5203dcefa878).
