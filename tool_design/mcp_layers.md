<https://engineering.block.xyz/blog/build-mcp-tools-like-ogres-with-layers>

Based on the article "Build MCP Tools Like Ogres... With Layers," the Layered MCP Server Pattern is a design strategy that moves away from creating single, monolithic tools. Instead, it advocates for breaking down functionality into distinct, cooperative layers. This approach structures the interaction, guiding the AI model through a logical sequence of discovery, planning, and execution, much like peeling back the layers of an onion.

The core problem this pattern solves is AI confusion. Giving an LLM access to a vast, complex API surface through a single tool is like asking a new employee to learn everything about a company by browsing its entire Slack history—it's overwhelming and prone to error. The layered pattern provides the necessary scaffolding to make the interaction reliable and predictable.

Here is a breakdown of the pattern and its layers, with examples from the financial services domain.

### The Three Core Layers of the MCP Server Pattern

#### Layer 1: The Discovery Layer
*   **Purpose:** Service and capability discovery. This layer answers the question, "What is possible?"
*   **Function:** It exposes a simple tool that allows the LLM to query the available services or domains of functionality. It doesn't execute any actions but provides metadata about what the system can do at a high level. This helps the AI orient itself and select the correct domain for a user's request.
*   **Example Tool:** `get_financial_service_info`
    *   **Description:** "Provides information about available financial services. Call this first to understand which service to use for tasks related to trading, account management, or payments."
    *   **Input:** `service_name: string` (e.g., "Trading", "Accounts", "Transfers")
    *   **Output:** A list of available high-level methods for that service (e.g., for "Trading", it might return: "[\"get_quote\", \"execute_trade\", \"get_positions\"]").

#### Layer 2: The Planning Layer
*   **Purpose:** Request formulation and validation schema. This layer answers the question, "How do I correctly ask for what I want?"
*   **Function:** Once the AI has discovered a capability, this layer provides the specific schema, parameters, required fields, and validation rules for a particular method. It gives the AI a precise template for building a valid request *before* attempting to execute it.
*   **Example Tool:** `get_method_schema`
    *   **Description:** "Gets the required parameters and schema for a specific financial service method. You must call this before executing a request."
    *   **Input:** `service_name: string` (e.g., "Trading"), `method_name: string` (e.g., "execute_trade")
    *   **Output:** A JSON schema detailing the required inputs for `execute_trade`, like: `{"account_id": "string", "ticker_symbol": "string", "order_type": "enum['market', 'limit']", "quantity": "number"}`.

#### Layer 3: The Execution Layer
*   **Purpose:** Performing the action. This layer answers the command, "Do the thing."
*   **Function:** This is the only layer that actually performs a state-changing operation or retrieves live data. It's a single, unified tool that takes the well-structured request—formulated with help from the Planning Layer—and executes it.
*   **Example Tool:** `execute_financial_request`
    *   **Description:** "Executes a specific, validated financial request. Always use `get_method_schema` first to ensure the request object is correctly formatted."
    *   **Input:** `service_name: string`, `method_name: string`, `request_payload: object`
    *   **Output:** The result of the API call, such as a trade confirmation or an error message.

---

### Example Workflow in Action

Imagine a user asks: **"Buy 10 shares of AAPL in my main brokerage account."**

1.  **AI Action 1 (Discovery):** The LLM first calls `get_financial_service_info(service_name="Trading")` to see what trading functions are available. It sees `"execute_trade"` is a valid method.
2.  **AI Action 2 (Planning):** The LLM then calls `get_method_schema(service_name="Trading", method_name="execute_trade")` to learn how to structure the request. It receives the schema requiring `account_id`, `ticker_symbol`, `order_type`, and `quantity`.
3.  **AI Action 3 (Execution):** The LLM can now formulate a perfect request. It calls `execute_financial_request` with the following payload:
    ```json
    {
      "service_name": "Trading",
      "method_name": "execute_trade",
      "request_payload": {
        "account_id": "acc_..._brokerage",
        "ticker_symbol": "AAPL",
        "order_type": "market",
        "quantity": 10
      }
    }
    ```

### Benefits of the Layered Pattern

*   **Reduces AI Errors:** By guiding the LLM step-by-step, it dramatically reduces the chance of malformed requests. The AI doesn't have to guess or infer complex schemas from a single, massive tool definition.
*   **Manages Complexity:** It allows developers to expose hundreds of API endpoints through just a few (e.g., three) MCP tools, making the system vastly more manageable for both the AI and the developers.
*   **Improves Debuggability:** When a failure occurs, it's easier to pinpoint where it happened. Did the AI fail at discovery (Layer 1), planning (Layer 2), or execution (Layer 3)?
*   **Conserves Context Tokens:** Instead of loading the schemas for all 200+ endpoints into the initial prompt, the AI only requests the schema for the one specific action it needs, leading to significant token savings.
*   **Modular and Maintainable:** Each layer has a distinct responsibility, making the server easier to maintain, test, and extend. Adding a new API service doesn't require rewriting a monolithic tool.