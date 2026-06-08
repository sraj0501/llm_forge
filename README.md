## 📚 FORGE SDK System Specification v1.0

### Project Codename: LLM\_Forge
**Mode:** Build Execution
**Goal:** To create a Meta-AI Orchestrator capable of unifying scattered developer knowledge (memory, skills, patterns) from existing AI tools (`opencode`, `Claude Code`, `Copilot`, `Codex`) into one portable system on Python/Go.

---

### 🛠️ I. Architectural Mandate & Constraints
1.  **Pillar:** The system must operate via a standardized **Universal Interface Layer (UIL)**, which abstracts all external tooling and APIs.
2.  **Language Choice:** Primary Orchestration Logic: **Python** (`forge_sdk/internal/core`). CLI and Wrapper Reliability: **Go**.
3.  **Portability Guarantee:** All code must be wrapped in a **Docker Container** to ensure zero dependency mismatch between Windows, Linux, and Cloud (WSL $\to$ Native Linux).
4.  **Data Constraint:** The system is *always* logging state changes. We treat output logs as continuous streams of transactional data, never final dumps, thus eliminating truncation issues by design.

---

### 🗺️ II. Core Schema Definitions (Blueprints)

#### A. The Forge Memory Database Schema
We will use a portable database (SQLite for MVP, PostgreSQL/JSONB for Cloud scale). All intelligence is stored here, *not* in ephemeral filesystem reads.

| Field | Data Type | Purpose | Source / Rule |
| :--- | :--- | :--- | :--- |
| **`record_id`** | UUID | Unique identifier for this knowledge event/pattern. | Auto-generated (Forge). |
| **`project_id`** | String | The owning scope/repository (e.g., `dev_utils`, `Aegis`). | Passed as Project Context. |
| **`asset_type`** | Enum | Catches the nature of the data: `SUCCESSFUL_PATTERN`, `SKILL_DEFINITION_SAMPLE`, `TOOL_USAGE_LOG`. | Defines *what* was saved. |
| **`timestamp`** | DateTime | When this event occurred. | System Clock (Auditability). |
| **`raw_input_context`** | JSONB (JSON) | The entire context received from the LLM/User prompt. MUST be logged for future reprocessing. | Mandatory capture of inputs. |
| **`skill_id`** | String | Reference to which Skill produced this memory. | Links the pattern back to its definition. |
| **`success_status`** | Boolean | Did the execution achieve its goal? | Operational Flag (Success/Failure). |
| **`detailed_output`** | Text (Large - BLOB) | The clean, structured, markdown-formatted result of the action. Handles large data volume without truncation risk. | Output from `ToolRunner`. |

#### B. Skill Graph Schema (`forge_skill_catalog.yaml`)
Every custom skill must conform to this template for discovery and execution.

```yaml
# Example placeholder: repo_inventory_scanner
skill_id: basic_repo_inventory
name: Repository Inventory Scanner
trigger_phrases: ["list directory contents", "check current files"]
description: |
  Used solely for diagnosing the internal structure of a repository context by listing visible directories and files. (Never to be used for code analysis.)
permissions:
  read: ["*"], 
  write: [], 
  exec: [ls, dir] # Defines permissible shell commands

required_inputs:
  - name: target_repository
    type: file_path
    is_required: true
    description: The directory to audit.
    default: null

execution_steps:
  - step_sequence: 1
    action_type: ToolExecution
    details:
      tool_name: shell
      command: ls -F
      arguments: [ "${target_repository}" ] # Variable substitution is key HERE!
```

---
### ✅ III. The Build Roadmap (Phases of Implementation)
This phased approach minimizes risk and provides early, measurable results. **Do not move to Phase II until Phase I is 100% functional.**

#### PHASE I: Proof-of-Concept - Local Inventory Scanner (MVP) [Target Goal: Validation]
*   **Goal:** Prove the entire stack works end-to-end on simple functionality.
*   **Focus Areas:** `basic_repo_inventory` Skill ID, The basic `git_wrapper`, and successful logging to `memory_db.py`.
*   **Testing Criteria:** 1. CLI runs successfully. 2. YAML loads without errors. 3. An actual file run generates output. 4. This output is correctly saved as a structured record in the local database for validation.

#### PHASE II: Core Dev Workflow Hook (Adding Depth)
*   **Goal:** Incorporating true development cycles and pattern recognition.
*   **New Skill ID Target:** `test_run_and_report`.
*   **Requirements Added:** The skill must execute test commands (`go test ./...` or `npm run build`) via the wrapper, collect stdout/stderr logs, check for non-zero exit codes, and standardize this complex result into the memory database.

#### PHASE III: Expanding Scope & Cross-Tool Integration (Complexity)
*   **Goal:** Integrating the "Intelligence" layer (LLM interaction).
*   **New Feature:** Implement a **Summarization Skill**. This skill takes raw logs from Phase II, feeds them to an LLM via the `api_client.py` wrapper, and commits the *summarized insight*, not just the raw log, to memory.

#### PHASE IV: Advanced Features & Production Readiness
*   **Goals:** Achieving true cross-system unification (Final Polish).
1.  **Remote Connection:** Integrate dynamic fetching of remotes/upstream status into the `git_wrapper`.
2.  **Cross-System Memory:** Implement a schema rule that tracks "Tool X failed here; try Tool Y next."
3.  **Deployment Target:** Refactor and optimize all code to be purely Docker-native, allowing seamless deployment from local machine to cloud environment (e.g., AWS/Azure container registry).

---
***This document serves as the contract for the project development. Every module built must refer back to this blueprint.***" 