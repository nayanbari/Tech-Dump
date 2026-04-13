Here is the AGENTS.md file formatted in a markdown code block so you can easily copy it for your copilot:
# AGENTS.md: Enterprise Backend Architecture & Implementation Guide
## 1. Core Technology Stack
 * **Language:** Python 3.12 or 3.13. **Strict Constraint:** Do not use Python 3.6, as it has reached end-of-life and lacks the security patches and async capabilities required for modern AI frameworks.
 * **Database:** PostgreSQL. Utilize JSONB fields for storing nested ISO 20022 schemas and complex LLM payloads. **Strict Constraint:** Do not use SQLite for the core application database, as it will encounter severe database locking and write-contention under concurrent enterprise loads.
## 2. Agent Orchestration Framework
 * **Framework Selection:** Implement the multi-agent recursive feedback loop using **LangGraph**. LangGraph is specifically designed for complex, stateful workflows with branching logic and provides the low-level control required for evaluator-refiner code generation loops.
 * **State Management:** Utilize LangGraph's built-in persistence layer to implement robust state checkpointing. Capturing complete agent states at strategic moments serves as an insurance policy against API timeouts during long-running tasks.
 * **Context Engineering:** Ensure the agent's context window only retains the *current, functional state* of the generated Python script and the immediate user feedback. Truncate historical, failed iterations to preserve API latency and model focus.
## 3. Secure Code Execution Sandbox
 * **Execution Isolation:** **Strict Constraint:** Never execute LLM-generated code using Python's native exec() or eval().
 * **Sandbox Infrastructure:** Use isolated microVM SDKs (such as **llm-sandbox** or **E2B**) to execute the generated Python scripts. This ensures code runs in isolated containers with no access to the host system, complete with network isolation and strict CPU/memory limits.
 * **Pre-installed Financial Libraries:** Pre-configure the execution sandboxes with specialized libraries to abstract XML complexity away from the LLM:
   * pain001: For automating the generation of ISO 20022-compliant payment files from CSV or dictionaries.
   * pyiso20022: For parsing, generating, and validating financial messages (e.g., pacs, camt) and outputting human-readable validation errors.
   * Faker: For high-speed generation of deterministic dummy text fields.
## 4. Atlassian Ecosystem Integration
 * **Jira API Integration:** Authenticate using OAuth 2.0 with the read:jira-work scope. To fetch multiple requirements or user stories at once for an epic, utilize the POST /rest/api/2/issue/bulkfetch endpoint.
 * **Confluence API Integration:** Use the Confluence Cloud REST API v2 to extract mapping tables and data dictionaries. To retrieve the raw, underlying XML representation of a Confluence page, query the GET /pages/{id} endpoint and append the body-format=storage query parameter.
## 5. High-Throughput Bulk Generation Pipeline
 * **Architecture:** Once the interactive chat loop yields an approved script, abstract the bulk data generation process into a **Producer/Consumer pattern**.
 * **Stages:**
   1. *Producer:* Dynamically generates queues of randomized inputs and parameters based on the final script.
   2. *Consumer-Producers (Workers):* Asynchronous task workers (e.g., Celery) execute the script locally. If LLM inference is still required for specific rows, enforce strict backpressure and a maximum queue size to prevent overwhelming the LLM API.
   3. *Consumer:* Validates the outputs against the targeted ISO 20022 XSD schema and streams the valid results into the final CSV or XML document for user download.
## 6. Compliance & Data Sovereignty
 * **Regulatory Alignment:** Ensure the platform's generation algorithms accurately mimic the statistical properties of production data. This directly addresses the Reserve Bank of India (RBI) mandates that require Regulated Entities to use synthetic data in non-production and testing environments to prevent live customer data leaks.
 * **Data Localization:** To comply with the RBI Master Direction on Data Localization, ensure that all cloud architecture (including PostgreSQL databases, Next.js frontend hosting, and sandbox microVMs) is physically provisioned within domestic Indian data centers.
