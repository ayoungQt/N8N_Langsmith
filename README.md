# N8N Automated Prompt Retraining Workflow

This project contains an n8n workflow designed to automate the process of retraining AI guard prompts based on feedback from LangSmith.

## Project Configuration

This section contains the critical IDs and names required to configure the n8n workflow.

### Google Drive

*   **Folder ID for Guard Prompts:** `1jplKoFeo3xLShJFO4_MpPUVHfDr-cOwx`

### Google Sheets

*   **Spreadsheet ID for Logging:** `1maCntXucpkj8MPV_w8Hi0NGvNka7r7aOvFTYilaqtis`
*   **Sheet Name for Logging:** `Guard_Retraining_Log`

### Guard Prompt File Names

This list maps the `guardName` used in Langsmith to the corresponding `.txt` file name in the Google Drive folder.

| Langsmith Guard Name                 | Google Drive File Name                  |
| ------------------------------------ | --------------------------------------- |
| `tool_supevisor`                      | `tool_supevisor_node.txt`               |
| `human_assistant_guard_agentic`        | `human_assistant_guard_agentic.txt`     |
| `unusual_prompt_agentic`               | `unusual_prompt_agentic_prompt.txt`     |
| `agentic_knowledge_base`               | `agentic_knowledge_base.txt`            |
| `qa_bot`                               | `qa_bot_node.txt`                       |
| `financial_advice_guard_agentic`       | `financial_guard_prompt.txt`            |
| `system_instruction_agentic`           | `system_instructions.txt`               |
| `conversational_hallucination_agentic` | `hallucination_guard_prompt.txt`        |

## N8N Architecture Notes

### LangChain Node Structure

This n8n instance uses a specific architecture for LangChain nodes. It is critical to follow this pattern for all LLM calls to prevent "dead nodes" and compatibility errors.

1.  **Model Node:** A separate `Google Gemini Chat Model` node (`@n8n/n8n-nodes-langchain.lmChatGoogleGemini`) must be used to define the LLM and its credentials. This node acts as the "brain".
2.  **Chain Node:** The `Basic LLM Chain` node (`@n8n/n8n-nodes-langchain.chainLlm`) is used to define the prompt and any other parameters for a specific call.
3.  **Connection:** The Model Node must be connected to the Chain Node using the special `ai_languageModel` input. Multiple Chain Nodes can be connected to a single Model Node.

This structure is different from older n8n versions where the model was configured directly within the chain node. **Do not use the deprecated, self-contained LLM nodes.**

## Configuring HTTP Request Nodes

When setting up an `HTTP Request` node in n8n, especially for APIs that require query parameters (like the Langsmith API), it is crucial to enter the parameters in the correct section of the node's properties panel.

**Correct Method:**

1.  Navigate to the **"Options - Add"** section within the node's properties.
2.  Select **"Query Parameters"** from the dropdown menu.
3.  Enter the key-value pairs for your parameters in the UI fields that appear. Use n8n expressions `{{...}}` in the value fields to pass in data from previous nodes dynamically.

**Incorrect Method:**

*   Do **NOT** use the "JSON/RAW Parameters" option for sending simple query string parameters. This option is for sending a raw JSON body with a `POST` or `PUT` request and will not work for `GET` requests expecting URL query parameters.

Following this will ensure the API receives the parameters correctly formatted in the URL (e.g., `https://api.example.com/data?key1=value1&key2=value2`).

## Key Lessons & Gotchas

This section documents critical lessons learned during the development and debugging of this workflow.

1.  **LangSmith API - Fetching Manual Feedback:**
    *   The `/feedback` endpoint returns automated evaluation data (e.g., `overall_cx_score`) by default.
    *   To get only the manual feedback entered in the UI, you **must** use the query parameter `feedback_source_type=app`.

2.  **LangSmith API - Case Sensitivity:**
    *   The values for feedback keys are **case-sensitive**. For example, the API will not find `false positive` if you search for `False Positive`. Always match the case exactly as it appears in the LangSmith UI.

3.  **n8n Looping - Passing Data:**
    *   When using a `SplitInBatches` node to loop, data generated *before* the loop may not be available to nodes *after* it.
    *   To solve this, you must enable the `passAlong` option in the `SplitInBatches` node's parameters. This ensures all original data is passed through each iteration of the loop.

4.  **Internal Workflow Filtering:**
    *   The LangSmith `/feedback` REST API does not support filtering feedback records by their `value` (e.g., you can query for the `key` "trip_validity", but you cannot ask for only the ones where the `value` is "false positive").
    *   The correct strategy is to fetch all feedback for a given guard and then use an `If` node within the workflow to perform the specific filtering. This is more robust than relying on complex, unsupported API queries. 