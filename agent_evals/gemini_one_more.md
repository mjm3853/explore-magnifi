# Prompt to Gemini:

can you simplify one more time, we really want to start simple and be able to add additional criteria and metrics as we go

... few more interactions...


# Gemini Response:

Yes, absolutely. It's very common for a single user turn to necessitate multiple tool calls by the agent to gather all the required information. The structure can be easily adapted to support this.

Here's an updated structure to handle multiple tool calls per conversation turn:

**Dataset Structure Supporting Multiple Tool Calls Per Turn:**

1.  **`user_query`**:
    *   What the user actually said.
    *   *Example*: "What's the current price of Google, its P/E ratio, and any recent news about them?"

2.  **`expected_tool_calls_sequence`**:
    *   A **list** of expected tool calls. Each item in the list represents a distinct tool call the agent should make.
    *   The order in the list can be important if the agent's reasoning implies a sequence (e.g., get an ID from one tool, then use it in another). If the order doesn't strictly matter, you can note that in your evaluation logic (treat it as a set).
    *   Each item in the list will be an object containing:
        *   **`tool_name`**: The specific name of the tool/API/function.
        *   **`tool_inputs`**: A dictionary/JSON object for the key parameters and their expected values for this specific tool call.
    *   *Example*:
        ```json
        [
          {
            "tool_name": "get_stock_price",
            "tool_inputs": {"ticker": "GOOG"}
          },
          {
            "tool_name": "get_company_financial_metric",
            "tool_inputs": {"ticker": "GOOG", "metric": "P/E_ratio"}
          },
          {
            "tool_name": "search_company_news",
            "tool_inputs": {"company_name": "Google", "time_filter": "last_24_hours"}
          }
        ]
        ```

3.  **`essential_answer_facts`**:
    *   A list of the critical pieces of information that *must* be in the agent's final response, drawing from the (expected successful) outputs of *all* the tool calls.
    *   *Example*: `["GOOG current price is $X", "GOOG P/E ratio is Y", "Recent news mentions Z event"]`

**Why This Structure Works for Multiple Tool Calls:**

*   **Explicitly Lists All Expected Actions:** `expected_tool_calls_sequence` clearly defines each step the agent should take.
*   **Captures Tool-Specific Inputs:** Each tool call in the sequence has its own defined inputs.
*   **Allows for Order (If Necessary):** The list structure naturally supports sequenced actions.
*   **Comprehensive Output Check:** `essential_answer_facts` ensures that the information gathered from all necessary tool calls makes it into the final response.

**Example Test Case (Multiple Tool Calls):**

*   **`user_query`**: "Compare the current stock price of MSFT and AAPL. Also, what are their latest earnings per share?"
*   **`expected_tool_calls_sequence`**:
    ```json
    [
      {
        "tool_name": "get_stock_price",
        "tool_inputs": {"ticker": "MSFT"}
      },
      {
        "tool_name": "get_stock_price",
        "tool_inputs": {"ticker": "AAPL"}
      },
      {
        "tool_name": "get_company_financial_metric",
        "tool_inputs": {"ticker": "MSFT", "metric": "EPS_latest_quarter"}
      },
      {
        "tool_name": "get_company_financial_metric",
        "tool_inputs": {"ticker": "AAPL", "metric": "EPS_latest_quarter"}
      }
    ]
    ```
*   **`essential_answer_facts`**: `["MSFT current price is $A", "AAPL current price is $B", "MSFT latest EPS is $C", "AAPL latest EPS is $D"]`

**Evaluation Process with This Structure:**

1.  **Feed `user_query` to the agent.**
2.  **Capture the agent's actual sequence of tool calls:**
    *   This should be a list of (tool\_name, tool\_inputs) pairs that the agent executed.
3.  **Compare the actual tool call sequence with `expected_tool_calls_sequence`:**
    *   **Presence and Correctness of Tools:** Are all expected tools called? Are there any unexpected tool calls?
    *   **Input Accuracy:** For each corresponding tool call, do the `tool_inputs` match?
    *   **Order (if important):** Is the sequence of calls correct? (This might be a stricter or more lenient check depending on your agent's design).
    *   Platforms like Vertex AI Agent Evaluation often provide ways to specify and evaluate sequences of tool calls, including checking parameters for each.
4.  **Assess `essential_answer_facts`** in the agent's final response against the expected outputs from the (correctly executed) tool chain.

This structure gives you a robust way to define and evaluate more complex agent behaviors where multiple tools must work in concert to satisfy a single user request. It remains relatively simple to annotate while providing deep insights into the agent's trajectory.