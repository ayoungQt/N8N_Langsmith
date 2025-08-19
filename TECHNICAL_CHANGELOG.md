### 2025-08-19

- feat(analysis): Aggregate multiple feedback examples per guard into numbered lists in `Final Assemble Case File` (`user_input`, `bot_error`) so downstream analysis sees all cases
- feat(audit): Appended evidence footer to LLM 1 prompt; model now outputs `{"examples_used":[...],"total_examples":N}` to prove multi-example coverage
- chore(models): Tuned temperatures (LLM 1 ≈ 0.2 for deterministic analysis; LLM 2 ≈ 0.3–0.4 for refinement)
- fix(duplication): Removed model fan-out (no data line into model nodes); ensured 1 item per guard end-to-end (LLM 1 → LLM 2 → Docs → Sheets)
- fix(docs): Title pulls guard file name from upstream (no `[0]`), adds ISO date; verified 3 unique docs for 3 guards
- fix(sheets): Hyperlink formula with USER_ENTERED input mode and per-item `documentId`/`id`; per-row guard name expression fixed (no `[0]`)
- docs(prompts): Kept Case File section intact (Verdict/Category/Reasoning/Analyst’s Note); appended instructions to cite multiple numbered examples

### 2025-08-18

- feat(workflow): consolidate multi-feedback per guard into a single case file; aggregate user_input and bot_error across runs
- feat(workflow): decode original guard prompt to text via `Reformat Original Guard` and feed into LLM 1
- fix(google-drive): ensure one file per guard by filtering name and folder, Return All off, Limit 1; verified 3 guards found and downloaded
- fix(expressions): switch from node-indexed `$items(...)[0]` to per-item `$('Node').item` references across nodes to respect n8n auto-looping
- fix(LLM): replace legacy false_positive fields with `verdict`, `false_positive_category`, and `reasoning`; updated LLM 1 prompt body accordingly
- fix(LLM): remove hardcoded guard label in report headers; make guard name dynamic from `guardApiName`
- chore(validation): added temporary debug fields (currentGuard, itemIndex, uniqueGuardNames) to validate grouping; removed after verification

### YYYY-MM-DD

-   **refactor(workflow):** Rearchitected the core data processing logic to group feedback by guard name before processing.
-   **feat(workflow):** Replaced the one-item-at-a-time loop with a more efficient data aggregation step using a `Code` node. This ensures all feedback for a single guard is consolidated.
-   **fix(expressions):** Updated numerous expressions throughout the workflow to correctly map and join data from the new grouped data structure (e.g., combining multiple `reasoning` texts into one).
-   **fix(looping):** Diagnosed and identified the root cause of the workflow processing the same guard multiple times; expressions were incorrectly referencing a node outside the main processing loop.

### Technical Change Log

**Date:** July 20, 2024

#### Core Logic & Bug Fixes
*   **fix(n8n-syntax):** Replaced all instances of the outdated `$item` variable with the correct `$input.item` in modern `Code` nodes.
*   **fix(n8n-syntax):** Replaced the legacy `$nodes` variable with the modern `$('Node Name').item` syntax for referencing data from previous nodes.
*   **fix(data-structure):** Corrected a critical data handling error in the `Find Guard File Name` node. The code now correctly processes the incoming array of `runs`, creates a new, valid JSON object, and populates it with `guardFileName` and `runDetails`, preventing the "json property must be an object" crash.
*   **fix(gdrive-search):** Re-architected the file search mechanism to be robust. It now uses a two-step process: a `Google Drive: Search` node to find files by name, followed by a `Code` node to filter the results and select the one file from the correct parent folder. This bypasses the limitations of the node's built-in filters.
*   **fix(download):** Configured the `Get Original Guard Prompt` (Google Drive) node to correctly use the `Download` operation and reference the file `id` from the output of the new filter node.

#### Node Refactoring
*   **refactor(workflow):** Replaced multiple legacy `Function` nodes with modern `Code` nodes to ensure compatibility and use up-to-date syntax.
*   **refactor(workflow):** Cleaned up workflow logic by removing a confusing and unnecessary `Build Run URL` node left over from a previous, flawed design.

### Technical Change Log

**Date:** July 19, 2024

#### Bug Fixes & Refinements
*   **fix(workflow):** Repaired the `Check for Records` IF node to be compatible with the user's n8n version. The condition now correctly uses the `notEqual` operator and checks against the array's `.length` property directly without a failing `.toNumber()` call.
*   **fix(gdrive):** Corrected the `Find Guard Prompt By Name` node. The operation was changed from the unsupported `search` to `List`, and a specific query was added to find the file by its exact name within the correct parent folder.
*   **fix(api):** Updated the `Get Initial Feedback` node's query parameters to include `key: "trip_validity"` and `score: 1`, allowing for more efficient filtering at the API source.

#### Refactoring
*   **refactor(workflow):** Removed the unnecessary `Prepare Guard Name` node entirely to improve clarity and reduce complexity. The logic was moved directly into the `Config` node, which now explicitly defines a `guardApiName` for each entry, making the data flow more transparent and robust.

---
### Technical Change Log

**Date:** 2025-07-18

#### Workflow Architecture
*   **feat(error-handling):** Refactored error handling into a dedicated, global workflow.
    *   Created `flows/error-workflow.json` with an `Error Trigger` node connected to a `Gmail` node for notifications.
    *   Modified the main workflow to remove the integrated `Gmail` node and set the `errorWorkflow` property in its settings to point to the new error handler. This provides more robust and centralized error management.
*   **feat(dynamic-processing):** Re-architected the main workflow to be data-driven and scalable for multiple guards.
    *   Created `flows/dynamic-main-workflow.json`.
    *   Replaced the static `Set` node with a `Code` node (`Guard Config List`) that outputs a JSON array of guard objects (`guardName`, `fileName`).
    *   Inserted a `SplitInBatches` node (configured for `batchSize: 1`) to loop through the array of guards.
    *   Updated all subsequent node expressions to use dynamic data from the loop's current item (e.g., `{{ $json.guardName }}`).

#### Bug Fixes & Refinements
*   **fix(api):** Corrected the `Get False Positive Feedback` HTTP Request node to successfully query the Langsmith API.
    *   Moved date generation expressions (`$now.minus(...)`) from the config node directly into the query parameters to ensure correct evaluation.
    *   Corrected n8n expression syntax by removing leading `=` and trimming whitespace.
    *   Updated query parameters to correctly filter by `key: "trip_validity"` and `value: "False Positive"` based on analysis of the API data structure.
    *   Removed the hardcoded `sendHeaders` and `headerParameters` properties to allow for standard credential selection from the UI without manual deletion.
*   **fix(compatibility):** Resolved multiple n8n version compatibility issues.
    *   Replaced the `Loop Over Items` node with the backward-compatible `SplitInBatches` node to fix the "dead node" issue on import.
    *   Corrected the output connection on the `SplitInBatches` loop node, changing it from the `done` output to the iterative output.
*   **fix(type-safety):** Resolved a type error in the `Check for Records` IF node.
    *   Modified the condition expression from `{{ $json.length }}` to `{{ $json.length.toNumber() }}` to ensure the value is treated as a number for comparison.
*   **chore(cleanup):** Identified obsolete workflow files for deletion to clean up the project directory.

#### Documentation
*   **docs(setup):** Updated `GUARD_RETRAINING_SETUP.md` to reflect the latest changes.
    *   Revised import instructions to include both `dynamic-main-workflow.json` and `error-workflow.json`.
    *   Added a new section explaining how to configure the list of guards in the `Guard Config List` code node.

---
# Changelog

## [Unreleased] - 2024-07-17

### Features
- **feat(workflow):** Implemented a fully dynamic looping system to process all 8 configured guards automatically without manual intervention.
- **feat(workflow):** Added a robust error handling branch that sends a detailed failure notification via a Gmail API HTTP request.

### Fixes
- **fix(api):** Corrected the core Langsmith API interaction logic. The workflow now fetches feedback first, then loops through results to get individual run details, correctly retrieving the necessary `run_name`.
- **fix(api):** Updated the Langsmith API filter `key` to the correct value `trip_validity` and the `value` to `False Positive` to accurately retrieve records.
- **fix(workflow):** Implemented an expression to automatically replace spaces with underscores in guard names to ensure API compatibility.
- **fix(gdrive):** Replaced the use of hardcoded Google Drive file IDs with a dynamic search for files by name, making the workflow more resilient.
- **fix(config):** Updated the list of `guardName` and `guardFileName` values in the central `Config` node to match the exact names used in your Langsmith and Google Drive environments.
- **fix(workflow):** Addressed compatibility issues by explicitly setting the `mode` to `mergeByPosition` on two Merge nodes ("Combine Data" and "Prepare for Final Refinement") to ensure predictable behavior on your n8n version.
- **fix(workflow):** Repaired the data flow by correctly connecting the `LLM 2: Refine Prompt` node to the `Create Google Doc` node.

### Refactoring
- **refactor(workflow):** Eliminated a redundant Merge node by updating downstream nodes to pull configuration data directly from the main `Config` node, simplifying the workflow's logic.
- **refactor(security):** Removed hardcoded API keys and IDs from all project files, replacing them with placeholders or environment-aware configurations.

### Chore
- **chore(project):** Deleted numerous obsolete and superseded workflow files to clean up the repository.
- **chore(docs):** Created a `README.md` file to serve as a central location for critical project information, including guard name mappings and configuration details. 