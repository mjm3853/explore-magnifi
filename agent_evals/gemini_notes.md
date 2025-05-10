# Prompt to Gemini:

I want to learn how to create a dataset (or datasets) that allow me to evaluate an Agentic AI system whose purpose is to interact with users about their financial portfolio and about general market information as well as knowledge base content. what is the best way to structure and annotate a dataset that would allow for evaluating the Agentic AI system?

# Gemini Response:

Okay, this is a fantastic and crucial task for building a robust financial agent. Evaluating an agentic system, especially in a sensitive domain like finance, requires a multifaceted dataset.

Here's a breakdown of how to structure and annotate such a dataset, keeping in mind the agent's diverse capabilities:

**I. Core Principles for Dataset Design:**

1.  **Diversity:** Cover a wide range of user intents, financial instruments, market conditions, KB topics, and levels of complexity.
2.  **Realism:** User queries should reflect how real users might ask questions (typos, ambiguity, colloquialisms).
3.  **Ground Truth Accuracy:** All factual data (portfolio, market, KB) must be accurate and, where time-sensitive, timestamped or clearly referenced.
4.  **Actionability:** Annotations should directly support the evaluation metrics you care about (e.g., intent accuracy, slot filling F1, response quality, task completion).
5.  **Modularity:** Consider creating separate but linkable datasets for different components (e.g., user profiles, KB content).

**II. Dataset Components & Structure:**

You'll likely need a few interconnected datasets:

**A. Main Interaction Dataset (Evaluation Set):**
This is where you'll capture user-agent interactions. Each row could represent a single turn or a complete conversation.

*   **Fields:**
    *   `interaction_id`: Unique ID for the interaction/conversation.
    *   `turn_id`: Unique ID for this specific turn within the conversation.
    *   `user_id`: (Optional, links to User Profile Dataset) To simulate personalized portfolio interactions.
    *   `timestamp`: Time of interaction (crucial for market data context).
    *   `user_utterance`: The raw text input from the user.
    *   `conversation_history`: (For multi-turn) A list of previous user/agent turns in this interaction.
    *   `expected_user_intent`: (Annotation) Categorized intent (e.g., `get_portfolio_value`, `ask_stock_performance`, `explain_kpi`, `compare_stocks`, `market_news_summary`).
    *   `expected_entities`: (Annotation) List of key entities and their types (e.g., `{"ticker": "AAPL", "metric": "P/E ratio", "date_range": "last_month"}`).
    *   `required_tools_or_knowledge_sources`: (Annotation) Which tools/APIs/KB sections *should* the agent use? (e.g., `portfolio_api`, `market_data_api:get_price`, `kb_search:inflation`).
    *   `ideal_agent_response_summary`: (Annotation) A concise summary of what the agent should convey.
    *   `ideal_agent_response_detailed`: (Annotation) A human-authored gold-standard response.
    *   `supporting_data_references`: (Annotation) IDs or pointers to the specific portfolio data, market data snapshot, or KB articles that ground the ideal response.
    *   `expected_action_type`: (Annotation, for agentic part) e.g., `information_retrieval`, `data_visualization_request`, `clarification_question`, `no_action_needed`.
    *   `complexity_rating`: (Annotation, e.g., 1-5) How complex is this query?
    *   `ambiguity_level`: (Annotation, e.g., low, medium, high) How ambiguous is the user query?
    *   `domain_tags`: (Annotation, e.g., `portfolio_specific`, `market_general`, `kb_definition`, `news_related`).
    *   `criticality`: (Annotation, e.g., low, medium, high) How critical is it to get this query exactly right (e.g., a query about selling stock is more critical than defining "P/E ratio").
    *   `safety_flags_triggered`: (Annotation) If the query could lead to harmful advice (e.g., asking for guaranteed returns, asking for non-public info). Mark if the ideal response should include disclaimers.

**B. User Profile & Portfolio Dataset (Synthetic or Anonymized):**
Used when `user_id` is present in the Interaction Dataset.

*   **Fields:**
    *   `user_id`: Unique ID for the user.
    *   `portfolio_snapshot_id`: If portfolios change over time, version them.
    *   `portfolio_holdings`: (JSON or structured)
        *   `[{ "ticker": "MSFT", "quantity": 100, "purchase_price": 150.00, "purchase_date": "YYYY-MM-DD" }, ...]`
    *   `cash_balance`: Current cash.
    *   `risk_profile`: (Optional, e.g., conservative, moderate, aggressive).
    *   `financial_goals`: (Optional, e.g., retirement, buying a house).
    *   `preferences`: (Optional, e.g., preferred news sources, sectors of interest).

**C. Knowledge Base (KB) Dataset:**
The content your agent can refer to.

*   **Fields:**
    *   `kb_article_id`: Unique ID for the article.
    *   `title`: Title of the article.
    *   `content`: The actual text content.
    *   `keywords`: Relevant keywords for search.
    *   `category`: (e.g., `financial_terms`, `investment_strategies`, `market_analysis_tools`).
    *   `source`: (e.g., `Investopedia_cleaned`, `internal_faq`).
    *   `last_updated`: Timestamp.

**D. Market Data Snapshot Dataset (Optional but Recommended for Reproducibility):**
If your agent uses real-time or recent market data, you need to "freeze" it for consistent evaluation.

*   **Fields:**
    *   `market_snapshot_id`: Unique ID.
    *   `timestamp_start`: Start of the period this snapshot covers.
    *   `timestamp_end`: End of the period.
    *   `data`: (JSON or structured)
        *   `{ "AAPL": { "price": 170.50, "volume": 2.5M, "change_pct": 0.015, "news_sentiment": "positive" }, ... }`
        *   General market indicators: `{"S&P500_index": 4500, "VIX": 15, ...}`

**III. Annotation Process - Best Practices:**

1.  **Clear Guidelines:** Develop a detailed annotation manual with definitions and examples for each annotation field (especially `expected_user_intent`, `expected_entities`, `required_tools_or_knowledge_sources`).
2.  **Multiple Annotators & Adjudication:**
    *   Have at least two annotators for a subset of the data.
    *   Calculate Inter-Annotator Agreement (IAA) (e.g., Cohen's Kappa).
    *   Have a senior annotator or domain expert adjudicate disagreements. This refines guidelines and improves consistency.
3.  **Iterative Refinement:** Start with a small batch, annotate, review, refine guidelines, then scale up.
4.  **Tooling:** Use annotation tools (e.g., Label Studio, Doccano, Prodigy, or even well-structured spreadsheets for simpler tasks) that can enforce schema and facilitate collaboration.
5.  **Domain Expertise:** Annotators should have a good understanding of finance. For `ideal_agent_response_detailed`, you might need financial professionals.
6.  **Focus on "Agentic" Aspects:**
    *   **Tool Selection:** For a query like "What's Apple's P/E ratio and how has its stock performed this year?", the `required_tools_or_knowledge_sources` might be `["market_data_api:get_pe_ratio:AAPL", "market_data_api:get_stock_history:AAPL:YTD"]`.
    *   **Reasoning Trace (Implicit):** The `supporting_data_references` and `ideal_agent_response_detailed` implicitly define the reasoning path. You could make this explicit by asking annotators to outline steps.

**IV. Data Generation Strategies:**

*   **Human-Crafted:** Domain experts write realistic user queries and ideal responses. High quality, but can be slow and expensive.
*   **Real User Logs (Anonymized):** If you have an existing system (even a basic one), logs can be a goldmine. Anonymize PII.
*   **Wizard-of-Oz:** A human simulates the AI behind the scenes, interacting with users. Capture these interactions.
*   **LLM-Generated & Human-Reviewed:**
    *   Use LLMs to generate diverse queries based on templates, seed questions, or personas.
    *   Use LLMs to generate initial "ideal responses."
    *   **CRITICAL:** Always have humans review, correct, and refine LLM-generated content, especially for financial accuracy and safety.
*   **Paraphrasing:** Take existing good examples and generate variations.
*   **Adversarial Examples:** Craft queries designed to trick or confuse the agent (e.g., ambiguous terms, conflicting information, out-of-scope requests).
*   **Scenario-Based Generation:**
    *   Define common financial scenarios (e.g., "User wants to rebalance portfolio," "User is concerned about inflation," "User wants to understand a specific company's recent earnings report").
    *   Generate multiple interaction flows for each scenario.

**V. How to Use This Dataset for Evaluation:**

1.  **Intent Classification & Entity Recognition:** Compare agent's predicted intent/entities against `expected_user_intent` / `expected_entities`. Metrics: Accuracy, Precision, Recall, F1-score.
2.  **Information Retrieval / Tool Use:**
    *   Check if the agent selected the `required_tools_or_knowledge_sources`.
    *   For KB queries, evaluate if the retrieved `supporting_data_references` are relevant. Metrics: Precision@K, Recall@K, MAP.
3.  **Response Quality:**
    *   **Automated:** ROUGE, BLEU, METEOR, BERTScore by comparing agent's response to `ideal_agent_response_detailed`. (Use with caution, as semantic similarity doesn't always mean factual correctness).
    *   **Human Evaluation:** Present agent's response and `ideal_agent_response_detailed` (or just the `ideal_agent_response_summary`) to human evaluators. Rate on:
        *   Accuracy (Factual correctness based on ground truth portfolio/market/KB)
        *   Completeness
        *   Clarity / Fluency
        *   Conciseness
        *   Helpfulness
        *   Safety (Absence of harmful advice, presence of necessary disclaimers)
        *   Grounding (Does the response logically follow from the cited sources?)
4.  **Task Completion (Multi-turn):** For multi-turn interactions, was the user's overall goal achieved? (Often a binary human judgment).
5.  **Failure Analysis:** When the agent fails, the rich annotations help pinpoint *why* (e.g., wrong intent, incorrect entity extraction, wrong tool used, hallucination).
6.  **Benchmarking:** Track performance on this dataset over time as you improve the agent.

**VI. Iteration and Maintenance:**

*   The financial world is dynamic. Your KB and potentially market data assumptions will need updates.
*   Continuously add new, challenging, and diverse examples to your evaluation set, especially based on observed failures in production or staging.

Creating this dataset is a significant undertaking, but it's the foundation for building a trustworthy and effective financial AI agent. Start small, iterate, and prioritize quality and diversity.