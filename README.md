# Multi-Agent HCI Research Assistant

## Abstract

This project implements a multi-agent deep research system designed to assist users in exploring the field of **Human-Computer Interaction (HCI)**. The system utilizes Microsoft AutoGen to orchestrate a team of four specialized agents: a **Planner**, a **Researcher**, a **Writer**, and a **Critic**. These agents collaborate in a round-robin conversation to break down complex queries, gather evidence from academic and web sources, synthesize findings with proper citations, and perform quality control. The system is powered by the **Qwen/Qwen3-8B** model and includes a dedicated safety layer to manage toxicity, personally identifiable information (PII), and off-topic queries. Users can interact with the system through a high-intensity Streamlit web interface or a command-line interface. Evaluation is conducted using an **LLM-as-a-Judge** framework across ten diverse test queries to measure relevance, clarity, and evidence quality.

---

## 1. System Design and Implementation

### 1.1 Research Focus

The system is optimized for **Human-Computer Interaction (HCI)** research. This includes topics such as user interface design, accessibility, augmented reality usability, and cognitive load in digital environments.

### 1.2 Agent Orchestration

The orchestration is managed in `src/autogen_orchestrator.py` using a `RoundRobinGroupChat` team. The workflow follows a structured sequence:

* **Planner**: Analyzes the query and creates an actionable research strategy.
* **Researcher**: Utilizes tools to gather data from web articles and academic databases.
* **Writer**: Synthesizes the gathered data into a structured report with inline citations.
* **Critic**: Reviews the output for accuracy and clarity. The Critic provides feedback for revision or issues a **TERMINATE** signal if the report meets quality standards.

### 1.3 Technical Tools

The Researcher agent has access to specific tools defined in `src/tools/`:

* **Web Search**: Wraps the **Brave Search API** to provide real-time web context.
* **Paper Search**: Utilizes the **Semantic Scholar API** to retrieve peer-reviewed academic abstracts and citation counts.
* **Citation Manager**: Ensures all sources are tracked and formatted correctly for the final bibliography.

### 1.4 Model Configuration

All agents and the judge module utilize the **Qwen/Qwen3-8B** model hosted on the Salt-Lab vLLM server. The system uses a specific `model_info` configuration to ensure compatibility with the AutoGen library.

---

## 2. Safety Design

Safety is a core component of the architecture, managed by a safety module that monitors all interactions.

### 2.1 Policy Categories

The system enforces a strict safety policy based on three primary categories:

* **Toxicity**: Blocking harmful, biased, or offensive language.
* **Personally Identifiable Information (PII)**: Redacting or blocking sensitive data like emails or phone numbers.
* **Off-Topic Control**: Ensuring the system remains focused on research tasks and refuses non-academic queries.

### 2.2 Implementation

The `SafetyManagerShim` in the orchestrator tracks these events. The Streamlit UI displays live metrics for "Safety Violations" and "Total Queries Checked" in the sidebar to maintain transparency for the user.

---

## 3. Evaluation Setup and Results

### 3.1 LLM-as-a-Judge

The evaluation pipeline in `src/evaluation/` uses a secondary instance of the LLM to act as an impartial judge. The judge evaluates the system on a scale of **0.0 to 1.0** based on the following criteria:

* **Relevance**: Does the response answer the specific query?
* **Evidence Quality**: Are the claims supported by cited sources?
* **Clarity**: Is the report well-organized and professional?
* **Factual Accuracy**: Are the findings consistent with the retrieved data?

### 3.2 Test Queries

The system was tested against **ten diverse queries** stored in `data/test_queries.json`. These include:

* Definitional queries regarding AR usability.
* Comparative queries on web versus mobile accessibility.
* Ethical considerations in healthcare AI.
* Adversarial queries designed to test safety guardrails.

### 3.3 Summary of Results

The latest evaluation run yielded the following performance metrics:

* **Average Overall Score**: Approximately **0.75 to 0.85**.
* **Safety Success**: 100 percent of off-topic or harmful queries were correctly identified.
* **System Reliability**: The inclusion of an asynchronous fallback in the judge ensures that the pipeline completes even if the API returns non-JSON text.

---

## 4. Discussion and Limitations

* **Multi-Agent Benefits**: The use of a separate Critic agent significantly improved the quality of citations compared to single-agent baselines.
* **Technical Hardships**: Managing asynchronous event loops in Python 3.13 proved challenging, requiring a custom approach to new event loop creation within the orchestrator.
* **Limitations**: The reliance on a single model family for both agents and the judge may introduce preference bias. Future versions should utilize a larger model for the judging phase to ensure higher rigor.

---

## 5. Setup and Reproducibility

### 5.1 Installation

1. Create a virtual environment: `python -m venv venv`.
2. Activate the environment: `.\venv\Scripts\activate` (Windows).
3. Install dependencies: `pip install -r requirements.txt`.

### 5.2 Running the System

* **Batch Evaluation**: `python src/evaluation/evaluator.py`.
* **Web Interface**: `streamlit run src/ui/streamlit_app.py`.

---

## References

* Microsoft AutoGen. (2024). *Multi-agent conversation framework.* [https://microsoft.github.io/autogen/](https://microsoft.github.io/autogen/)
* Semantic Scholar. (2024). *Official API Documentation.* [https://www.semanticscholar.org/product/api](https://www.semanticscholar.org/product/api)
* Zheng, L., et al. (2023). *Judging LLM-as-a-Judge with MT-Bench.* *NeurIPS 2023.*
