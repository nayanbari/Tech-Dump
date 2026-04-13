Here is the updated AGENTS.md file reflecting your strict constraints of using Python 3.6 and SQLite.
# AGENTS.md: Enterprise Backend Architecture & Implementation Guide (Legacy Stack)
## 1. Core Technology Stack
 * **Language:** Python 3.6. Note that this version reached its End of Life in December 2021 and no longer receives security or bug fixes.[1]
 * **Database:** SQLite. To handle concurrent users, you must enable Write-Ahead Logging (WAL) mode and configure a high lock timeout value to mitigate database-level locking bottlenecks.
## 2. Agent Orchestration Framework
 * **Framework Selection:** Modern orchestration frameworks like LangChain and LangGraph officially require Python 3.8.1 or higher and cannot be installed on Python 3.6. You must build a custom, manual orchestration loop for the evaluator-refiner code generation.[2]
 * **State Management:** Since you are building a custom loop, conversational state and script iterations must be manually saved to SQLite tables. Keep write transactions extremely brief to avoid locking out other agents.
## 3. Secure Code Execution Sandbox
 * **Execution Isolation:** You will need to build custom Docker containers or rely on compatible older sandbox environments that still support Python 3.6.
 * **Manual XML Construction:** Modern financial libraries like pain001 (requires Python 3.9+) and pyiso20022 (requires Python 3.8+) are incompatible with this stack. Your agent must be prompted to manually construct ISO 20022 XML using Python's built-in xml.etree.ElementTree module.
 * **Dummy Data:** Rely on older, compatible versions of libraries like Faker to generate dummy fields, ensuring you pin the dependency to a version that supports Python 3.6.
## 4. Atlassian Ecosystem Integration
 * **API Integration:** You will need to use older versions of the atlassian-python-api library or construct manual OAuth 2.0 requests using compatible older versions of the requests library.
 * **Confluence Extraction:** When querying the Confluence Cloud REST API v2, append the body-format=storage parameter to the /pages/{id} endpoint to retrieve the raw XML representation of the page.[3, 4]
## 5. High-Throughput Bulk Generation Pipeline
 * **Architecture:** Use a decoupled Producer/Consumer pattern.
 * **Database Protection:** To prevent severe write-contention on the single SQLite database file, worker processes should write the generated synthetic records to temporary local CSV files first, rather than inserting rows into SQLite concurrently.
## 6. Compliance & Data Sovereignty
 * **Data Localization:** To comply with the Reserve Bank of India (RBI) directive on the storage of payment system data, ensure that the SQLite database file and the server executing the Python 3.6 code are physically stored only in India.[5, 6]
 * **Security Posture:** Apply strict network isolation to the server to mitigate the risks of unpatched Python 3.6 vulnerabilities.[1]
