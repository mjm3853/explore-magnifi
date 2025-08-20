<https://useai.substack.com/p/mcp-tool-design-from-apis-to-ai-first>

Here's a summary of the article "MCP Tool Design: From APIs to AI-First Interfaces" formatted for easy digestion:

## MCP Tool Design: From APIs to AI-First Interfaces

The article discusses how designing **Model Context Protocol (MCP) tools for AI models** differs significantly from traditional API design.

### Key Challenges with Traditional APIs for AI:
*   AI models lack persistent learning and have limited exploration capabilities.
*   They rely on immediate context, not long-term comprehension.
*   Each AI interaction starts from scratch, requiring clear, self-contained information.

### The "AI-First" Design Philosophy:
*   Focus on **user intent** rather than internal system operations.
*   Tools should be designed around what a user wants to accomplish.
*   Every tool response, including errors, should guide the AI's next action.
*   Balance context with efficiency to manage token limits.

### Core Design Principles for Effective MCP Tools:
*   **Structured Descriptions**: Use XML tags like `<usecase>` (what it does/when to use) and `<instructions>` (how to use) for clarity.
*   **The 90/10 Rule for Descriptions**: Provide essential information upfront, and use intelligent error handling for edge cases.
*   **Prompt Injection in Responses**: Use tool responses (success or error) to suggest the next logical steps or correct usage to the AI.
*   **Consolidation Strategy**: Group API functionalities into fewer, intent-based MCP tools (e.g., transforming 80 API endpoints into ~12 MCP tools for Circle.so) to prevent "token explosion" and simplify AI reasoning. Keep "risky" operations separate.

By adopting an "AI-first" approach, developers can create tools that enable AI to perform complex, multi-step tasks naturally and efficiently, without needing to understand the underlying technical data model.

## The Final Tool Set
Those 80 Circle endpoints become ~12 intent-based MCP tools:

- Finding things: findContent, findMembers, findSpaces (replaces ~15 endpoints)
- Understanding activity: getSpaceActivity, getSpaceEvents (replaces ~20 endpoints)
- Taking action: publishContent, moderateContent, manageMembership (replaces ~25 endpoints)
- Communication: sendDirectMessage, bulkMessage (replaces ~10 endpoints)
-   Automation: watchForChanges, exportData (replaces ~10 endpoints)