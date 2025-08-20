<https://engineering.block.xyz/blog/blocks-playbook-for-designing-mcp-servers>

Here's a friendly markdown summary of "Block's Playbook for Designing MCP Servers":

## Block's Playbook for Designing MCP Servers

Block's engineering team shares their wisdom from building over 60 Model Context Protocol (MCP) servers, focusing on a **workflow-first design** for tools that work beautifully with Large Language Models (LLMs).

### The Big Idea: Workflow-First Design 🚀
Forget simply exposing every API endpoint! Instead, think about the entire user journey. Design high-level tools that combine multiple internal steps to help users achieve their goals efficiently, in as few steps as possible.

**For Example:**
*   Instead of separate tools for `get_user` and `upload_file`, create one `upload_file` tool that handles fetching user info internally.

### Key Tips for Great MCP Server Design:

*   **Name & Parameter Power!** ✨
    *   Treat tool names, descriptions, and parameters like prompts for your LLM – clarity is king!
    *   Use **Pydantic models** for complex parameters. They come with built-in descriptions and help with JSONSchema for accurate parsing.

*   **Be a Token Budget Boss!** 💰
    *   LLMs have limited "context windows" (token budgets).
    *   **Guard against huge outputs**: Have tools throw errors or paginate data if it's too big (e.g., a file over 400KB).
    *   Provide **actionable error messages** – tell the LLM exactly what went wrong and how to fix it!

*   **Cache Smartly!** ⚡
    *   Avoid including dynamic, ever-changing data (like timestamps) directly in your tool instructions. This helps LLMs cache prompts more effectively.

*   **Play to LLM Strengths!** 💪
    *   LLMs are great at:
        *   **SQL queries** (use tools like DuckDB for structured data).
        *   Generating **Markdown** or **Mermaid diagrams**.
    *   LLMs struggle with:
        *   Complex **multi-step planning** (try to reduce chained calls).
        *   Generating super strict **JSON formats** (Markdown or XML might be better).

*   **Security First: Auth & Permissions** 🔒
    *   Use **OAuth** for authentication.
    *   Store tokens securely in keyrings.
    *   Manage token lifecycles properly.
    *   Design tools with a **single risk level** (e.g., all read-only, or all non-read) to simplify user permissions. Consolidate similar read-only actions into one tool.

### Real-World Examples from Block:

*   **Google Calendar MCP**: Evolved from mirroring API endpoints to using **DuckDB for analytics** and a single `query_database` tool that accepts SQL queries.
*   **Linear MCP**: Transformed from many tiny tools to a more powerful `get_issue_info` with categories, and eventually to `execute_readonly_query` and `execute_mutation_query` tools that directly accept GraphQL queries, simplifying complex requests immensely.

