AGENTS.md
Project Overview
We are building an AI-Powered Synthetic Test Data Generator. The backend is a modular monolith built with Python 3.10+, FastAPI, SQLite, LangGraph for stateful Human-in-the-Loop (HITL) workflows, and E2B microVMs for secure, untrusted code execution.
Architectural Rules & Directory Structure
Adhere strictly to Domain-Driven Design (DDD) principles. The codebase is organized as follows:
• src/core/: Configuration, Pydantic settings, cryptographic setup.
• src/domain/: Pure business logic, Python entities, abstract interfaces. No FastAPI or ORM dependencies.
• src/data/: SQLAlchemy ORM models, SQLite database connection management, and Atlassian/OpenAI API clients.
• src/services/: Orchestration logic for E2B execution, prompt assembly, and Atlassian context summarization.
• src/api/v1/: FastAPI Presentation layer (routers, endpoints, dependency injection).
Core Technical Constraints
1. Database and Concurrency
• Database: SQLite (file-backed).
• Concurrency: Because FastAPI is asynchronous, SQLite must be configured to handle concurrent requests. Always initialize the SQLAlchemy engine with connect_args={"check_same_thread": False}.
• Journal Mode: Use Write-Ahead Logging (WAL) mode for better concurrent read/write performance.
2. Security and Authentication
• Passwords: Hash all user passwords using the Argon2 algorithm.
• Token Storage: Atlassian API tokens (Jira/Confluence) MUST NOT be stored in plain text. Use the cryptography library's Fernet symmetric encryption inside a custom SQLAlchemy TypeDecorator to automatically encrypt tokens before saving to SQLite and decrypt them upon retrieval.
• Sessions: Use JWTs for stateless authentication.
3. LangGraph Human-in-the-Loop (HITL)
• Use LangGraph to orchestrate the multi-step conversational flow.
• Use LangGraph's persistent checkpointer (e.g., MemorySaver or an SQLite checkpointer) mapped to a specific thread_id to persist the state when paused.
• Interrupts: To pause the graph and wait for human review of the generated preview data, use the interrupt() function. Do not use legacy input() functions.
• Resuming: To resume the graph after the user provides feedback via the FastAPI endpoint, use Command(resume=feedback_data) when re-invoking the graph.
4. Code Generation & Execution (E2B)
• Prompting: When prompting the LLM to write the generation script, explicitly force it to use faker (for realistic data) and pandas (for serialization).
• Execution: Execute the generated Python script securely using the E2B Python SDK. Use Sandbox.create() to initialize a Firecracker microVM, and execute the script using sandbox.run_code(script_content).
• File Extraction: After execution, read the generated CSV/JSON file out of the sandbox using sandbox.files.read('/path/to/file').
5. File Delivery
• Streaming: Never load large generated datasets entirely into memory. Instead, save the extracted E2B file to a temporary local directory.
• FastAPI Endpoint: Create an asynchronous generator that yields chunks of the file (e.g., 64KB blocks). Pass this generator to FastAPI's StreamingResponse.
• Headers: Always set Content-Disposition: attachment; filename="synthetic_data.csv" to force a browser download dialog.
