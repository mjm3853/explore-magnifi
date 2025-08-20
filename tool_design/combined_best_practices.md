Here's a combined, deduplicated summary of best practices from both articles, followed by a recommended playbook for designing Model Context Protocol (MCP) tools:

## Combined Best Practices for MCP Tool Design

Designing MCP tools for AI models requires a fundamental shift from traditional API design. The core philosophy is to enable the AI to understand and execute user intent efficiently, rather than exposing granular system operations.

### Overarching Philosophy: AI-First & Workflow-Centric

*   **Focus on User Intent/Workflow-First Design**: Instead of mirroring internal API structures, design tools around what a user (via the AI) wants to achieve. Combine multiple internal API calls into high-level, intent-based tools that complete a user workflow in as few steps as possible.
*   **Enable Natural Tool Chaining**: Design tools so their outputs naturally feed into the inputs of subsequent tools, facilitating complex, multi-step tasks without the AI needing deep understanding of underlying data models.

### Tool Definition & Interaction Principles:

*   **Clarity in Naming & Descriptions**: Treat tool names, descriptions, and parameters as critical prompts for the LLM.
    *   Use **structured descriptions** (e.g., XML tags like `<usecase>` for purpose and `<instructions>` for usage) to help the AI quickly grasp relevance and correct usage.
    *   For complex parameters, use **Pydantic models** to leverage descriptions and JSONSchema serialization for robust parsing.
*   **The 90/10 Rule for Descriptions**: Provide essential information upfront. Avoid overly exhaustive descriptions that consume valuable token budget.
*   **Smart Context & Token Management**:
    *   Be highly mindful of the LLM's context window.
    *   Implement **checks within tools** to prevent oversized outputs (e.g., large files) by throwing errors, truncating, or paginating.
*   **Actionable Responses & Prompt Injection**:
    *   Every tool response, whether success or error, is an opportunity to guide the AI's next action.
    *   **Error messages should be explicit**: clearly state what went wrong, why, what to do instead, and provide examples of correct usage. This serves as an iterative "teaching moment" for the AI.
*   **Leverage LLM Strengths & Avoid Weaknesses**:
    *   **Strengths**: Design tools for tasks LLMs excel at, like generating SQL queries for structured data (e.g., using DuckDB) or creating Markdown/Mermaid diagrams.
    *   **Weaknesses**: Avoid designing tools that require LLMs to perform multi-step planning with many chained calls or generate strict, complex JSON formats (Markdown or XML might be more forgiving).

### System & Operational Considerations:

*   **Consolidation Strategy**: Group API functionalities into fewer, intent-based MCP tools. This reduces the number of tools the AI considers, combats "token explosion," and simplifies the conceptual model for the AI. Keep distinct, "risky" operations (like delete) separate.
*   **Prompt Prefix Caching**: Avoid injecting highly dynamic data (like timestamps) into tool instructions to maximize the benefits of LLM prompt caching.
*   **Secure Authentication**: Implement robust authentication mechanisms (e.g., OAuth) and ensure tokens are stored securely (e.g., in keyrings) with proper lifecycle management.
*   **Permission Management**: Design tools with a consistent risk level (e.g., all read-only, or all write-enabled) to simplify permission management. Consolidate related read-only actions into a single tool where appropriate.

---

## Recommended Playbook for Designing MCP Tools

Here's a step-by-step playbook to guide the design of effective MCP tools:

### Phase 1: Conceptual Design & Strategy

1.  **Define User Workflows (AI-First Thinking):**
    *   **Goal:** Instead of listing API endpoints, identify common user tasks and the desired outcomes.
    *   **Action:** Map out end-to-end user workflows. Ask: "What does the user ultimately want to accomplish?"
    *   *Example: Instead of `createUser`, `uploadProfilePic`, think "Onboard New User."*

2.  **Consolidate by User Intent:**
    *   **Goal:** Create a minimal set of powerful, high-level tools.
    *   **Action:** Group related API functionalities into single MCP tools that fulfill a specific user intent. Avoid 1:1 mapping of APIs to tools.
    *   *Example: Combine `createPost`, `addTags`, `publishPost` into a single `manage_post` tool with different actions.*

3.  **Assess LLM Strengths & Weaknesses:**
    *   **Goal:** Leverage the LLM's capabilities and mitigate its limitations.
    *   **Action:** For each workflow, determine if the task aligns with what LLMs do well (e.g., text generation, structured data querying) or poorly (e.g., precise multi-step planning, strict JSON generation). Design tools accordingly.

### Phase 2: Tool Definition & Implementation

4.  **Craft Clear Tool Names & Descriptions:**
    *   **Goal:** Ensure the LLM immediately understands the tool's purpose and usage.
    *   **Action:**
        *   Choose intuitive, action-oriented tool names.
        *   Use structured descriptions (e.g., XML `<usecase>` and `<instructions>`) to clearly explain what the tool does and how to use it.
        *   Adhere to the "90/10 rule": provide essential info, not exhaustive detail.

5.  **Design Robust Parameters:**
    *   **Goal:** Provide clear, structured inputs for the AI.
    *   **Action:** Use Pydantic models for complex parameters. This allows for built-in descriptions, validation, and clear JSONSchema serialization, which benefits the AI's understanding.

6.  **Implement Smart Context & Token Management:**
    *   **Goal:** Prevent LLM context window overflow and improve efficiency.
    *   **Action:**
        *   Be concise in tool descriptions and parameter definitions.
        *   Build logic within tools to handle large outputs (e.g., truncate, paginate, or throw errors for oversized data).

7.  **Develop Actionable Responses & Error Handling:**
    *   **Goal:** Guide the AI's subsequent actions and enable learning from mistakes.
    *   **Action:**
        *   Every tool response (success or failure) should "inject" helpful information back into the LLM's context.
        *   Error messages must be specific: state what went wrong, why, and provide clear instructions or examples for correction.

### Phase 3: Operational & Security Readiness

8.  **Ensure Secure Authentication & Authorization:**
    *   **Goal:** Protect your systems and user data.
    *   **Action:** Implement robust authentication (e.g., OAuth). Store API keys/tokens securely. Design tools such that a single tool has a consistent risk level (e.g., all read-only).

9.  **Optimize for Caching:**
    *   **Goal:** Improve performance and reduce inference costs.
    *   **Action:** Avoid injecting dynamic, frequently changing data into static tool instructions to maximize prompt prefix caching.

By following this playbook, you can design MCP tools that are not only powerful but also intuitive and efficient for Large Language Models, unlocking new capabilities for AI-driven applications.