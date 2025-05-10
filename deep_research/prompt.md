# Deep Technical Research Prompt: Financial Services Chatbot

## Primary Goal
Deep technical research into architectures, algorithms, and implementation methodologies for a **financial services chatbot**. This chatbot must enable novice investors to naturally query their personal portfolio data and market data through a controlled, non-generative conversational interface.

---

## Source Prioritization
- **Academic papers**
- **Peer-reviewed journals**
- **Conference proceedings** (e.g., ACL, EMNLP, AAAI for NLP; relevant finance/HCI conferences)
- **Technical white papers**
- **Documented open-source projects or frameworks**

---

## Key Areas of Technical Investigation

> **Strictly within a non-generative, compliant framework**

### 1. NLP for Financial Queries & Dialogue Control
- **Intent Recognition & Slot Filling:**
  - Robust techniques for accurately parsing user utterances related to:
    - Portfolio performance (e.g., `"rate of return YTD"`, `"asset allocation for my ISA account"`)
    - Transaction history (e.g., `"show trades in AAPL last month"`)
    - Market data lookups (e.g., `"current price of GOOG"`, `"interest rate trends"`)
- **Deterministic Dialogue Management:**
  - Strategies for rule-based, finite-state, or structured graph-based dialogue flows
  - Focus on systems ensuring predictable conversational paths and manageable state tracking
  - Explicitly **avoid** unconstrained generative models for dialogue progression
- **Query Formulation:**
  - Methods for translating recognized intents and extracted entities into precise, executable queries against structured backend data (portfolio databases, market data APIs)

### 2. Compliant & Controlled Response Generation Systems
- **Template-Based & Retrieval-Augmented (Non-Generative) Responses:**
  - Technical solutions for dynamically populating pre-defined response templates or retrieving and presenting information from curated knowledge bases
- **Architectures for Auditable & Explainable Outputs:**
  - Design patterns that ensure all user-facing information is traceable to its source and generated through deterministic processes (crucial for legal, risk, and compliance oversight)
- **Personalization within Constraints:**
  - Techniques for tailoring informational and analytical responses (e.g., `"Your portfolio is X% in tech, which outperformed the market by Y% last quarter"`) using user data and market data **without** resorting to generative AI for synthesis

### 3. Secure Data Integration & Real-time Querying Architecture
- **Secure API Integration:**
  - Best practices for integrating with heterogeneous data sources (e.g., core banking systems, brokerage platforms, market data vendors via REST/GraphQL APIs, FIX protocols if relevant for data ingestion)
  - Focus on authentication, authorization, and data encryption in transit and at rest
- **Efficient Data Aggregation & Query Optimization:**
  - Techniques for real-time aggregation of portfolio and market data to support low-latency responses
  - Consider caching strategies, optimized database schemas, and efficient query execution plans
- **Scalable & Resilient System Design:**
  - Architectural considerations (e.g., microservices, message queues, stateless components) for building a system that can scale with user load and maintain high availability

### 4. Foundational Use Cases & Technical Implementation Roadmaps
- **Identify core MVP use cases** for novice investors (e.g., balance inquiry, simple performance review, asset breakdown) and map them to specific NLP tasks, dialogue structures, and data requirements
- **Research technical stacks, libraries, or components** (e.g., specific NLU engines suitable for controlled environments, data integration tools) that align with the non-generative and compliant mandate
- **Outline potential technical milestones** for prototyping and iterative development

---

## Expected Research Output
- Comparative analysis of NLP/NLU techniques suitable for constrained financial dialogue
- Proposed system architectures and design patterns (e.g., data flow diagrams, component interaction models)
- Enumeration of key technical challenges (e.g., ambiguity resolution in financial language, maintaining data consistency, ensuring response accuracy under compliance) and innovative solutions or mitigation strategies
- Examples of technical implementations or well-documented frameworks (even if from adjacent domains) that demonstrate relevant principles

---

## Strict Exclusions (for Filtering OUT Results)

> **Exclude any sources or content containing the following keywords:**

- `LLM_generative_responses`
- `chatbot_direct_investment_advice`
- `robo_advisor_portfolio_construction`
- `unsupervised_dialogue_generation`
- `end_to_end_deep_learning_chatbots` *(unless specifically focused on a highly controlled, explainable component within the larger system)*
- `marketing_collateral`
- `UX_design_trends` *(unless directly substantiating a technical implementation choice for interaction)*