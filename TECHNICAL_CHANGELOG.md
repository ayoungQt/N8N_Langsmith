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