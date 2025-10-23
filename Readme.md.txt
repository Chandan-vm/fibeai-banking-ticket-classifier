# FibeAI Banking Support Ticket Classifier Service

## Solution by: [Your Name]

Thank you for the engaging challenge! This document outlines the solution for the banking support ticket classification service. The goal is to triage incoming banking support tickets by classifying them as needing either an AI-generated code remediation or a Vibe-coded troubleshooting script, and providing an appropriate action plan.

---

## 1. Service Overview

*   **Objective:** To classify incoming banking support tickets based on `channel`, `severity`, and `summary` into one of three categories: "AI Code Patch", "Vibe-coded Troubleshooting Script", or "Manual Review Required". This aims to automate initial triage and accelerate response times.
*   **Approach:** A rule-based Python service (`TicketClassifier`) that leverages keyword matching and severity thresholds for classification. This approach was chosen for its clarity, interpretability, and ease of initial deployment.
*   **Output:** A structured JSON response containing the decision, the reasoning behind it, and a lightweight checklist of actionable next steps.
*   **Implementation Language:** Python 3.x (standard library only)

---

## 2. Core Logic (`ticket_classifier.py`)

The core logic for the classification service is encapsulated within the `TicketClassifier` class, located in the `ticket_classifier.py` script.

### `TicketClassifier` Class Structure:

*   **`__init__(self)`**:
    *   Initializes `self.rules`, a dictionary that serves as the configurable knowledge base for classification. These rules are easily modifiable and, in a production environment, could be externalized for dynamic updates.
    *   **`ai_code_patch_keywords`**: A list of terms indicative of technical, code-related problems (e.g., "bug", "api problem", "database error", "service outage").
    *   **`vibe_workflow_keywords`**: A list of terms associated with common customer service issues resolvable by predefined workflows (e.g., "password reset", "transaction dispute", "account access").
    *   **`severity_for_ai_patch`**: A list of severity levels (e.g., "high", "critical") that, when combined with matching keywords, prioritize an AI Code Patch.
    *   **`severity_for_vibe`**: A list of severity levels (e.g., "medium", "low") that, if not an AI patch, suggest a Vibe workflow.
    *   **`manual_review_ambiguous_keywords`**: Keywords that signal an ambiguous issue, potentially forcing a "Manual Review" even at high severity.

*   **`_check_keywords(self, text: str, keywords: list) -> list`**:
    *   This is a private helper method designed to efficiently identify all matching keywords from a given `keywords` list within a `text` string. It returns the list of found keywords, which is then used to construct detailed reasoning for the classification. This abstracts keyword matching logic for better readability and reusability.

*   **`classify_ticket(self, ticket_json: str) -> dict`**:
    *   **Input Handling:** Accepts a JSON string describing the support ticket (`{"channel": "...", "severity": "...", "summary": "..."}`). It includes robust error handling for invalid JSON input, returning an immediate error response if parsing fails.
    *   **Data Normalization:** Extracts `channel`, `severity`, and `summary` fields, converting them to lowercase to ensure case-insensitive matching against the defined rules.
    *   **Decision Prioritization (Hierarchical Logic):**
        1.  **AI Code Patch (Highest Priority):** The service first attempts to classify as "AI Code Patch". This occurs if the ticket's `summary` contains any `ai_code_patch_keywords` AND its `severity` is one of the designated `severity_for_ai_patch` levels ("high" or "critical"). This prioritizes urgent, system-critical technical issues.
        2.  **Vibe-coded Troubleshooting Script (Second Priority):** If the ticket does not qualify for an AI Code Patch, the service then checks for "Vibe-coded Troubleshooting Script". This occurs if the `summary` contains any `vibe_workflow_keywords` OR if the `severity` is one of the designated `severity_for_vibe` levels ("medium" or "low"). This covers most standard customer service interactions.
        3.  **Manual Review Required (Fallback & Specific Ambiguity):**
            *   This is the default classification if neither of the above conditions is met.
            *   A specific refinement ensures that if a ticket has `high` or `critical` severity *but* doesn't match AI code patch criteria, or if it contains `manual_review_ambiguous_keywords`, it is explicitly routed for "Manual Review". This ensures that critical but unclear issues receive immediate human expert attention.
    *   **Structured Output:** For each classification, the method constructs and returns a dictionary containing:
        *   `"decision"`: The determined classification.
        *   `"reasoning"`: A dynamically generated explanation, detailing which keywords or severity levels led to the decision.
        *   `"next_actions"`: A clear, step-by-step checklist tailored to guide the relevant team on the subsequent actions.

### Example Usage (from `ticket_classifier.py`):

To demonstrate the classifier's functionality and for local testing, the `if __name__ == "__main__":` block in `ticket_classifier.py` provides several test cases:

```python
# Example of how to use the classifier:
if __name__ == "__main__":
    classifier = TicketClassifier()

    # AI Code Patch - High Severity API Error
    ticket_ai = '{"channel": "web", "severity": "high", "summary": "Login API returning 500 internal server error for all users. Backend service down."}'
    print("\nTicket 1 (AI Code Patch Expected):")
    print(json.dumps(classifier.classify_ticket(ticket_ai), indent=2))

    # Vibe Workflow - Medium Severity Password Reset
    ticket_vibe = '{"channel": "phone", "severity": "medium", "summary": "Customer unable to reset password, link not arriving."}'
    print("\nTicket 2 (Vibe Workflow Expected):")
    print(json.dumps(classifier.classify_ticket(ticket_vibe), indent=2))

    # Manual Review - Ambiguous High Severity
    ticket_manual = '{"channel": "email", "severity": "high", "summary": "Customer complaining about general system slowness, unclear root cause."}'
    print("\nTicket 3 (Manual Review Expected - Ambiguous High):")
    print(json.dumps(classifier.classify_ticket(ticket_manual), indent=2))
    
    # (More examples are available within the script for comprehensive testing)

3. AI Tooling & Prompts Leveraged
This solution was developed with significant assistance from a Large Language Model (LLM, like the one you're currently interacting with). My role as the "data scientist" involved guiding the AI with specific prompts, integrating its suggestions, and refining the output for quality, domain relevance, and adherence to requirements. This demonstrates a practical application of AI-assisted development.
Key areas where the LLM proved instrumental:
Initial Code Structure Generation: Utilized prompts to rapidly scaffold the fundamental Python class (TicketClassifier), method signatures (classify_ticket, __init__), basic JSON parsing, and the initial if/elif/else decision logic. This significantly accelerated the boilerplate setup.
Keyword Brainstorming & Refinement: Leveraged the AI's extensive knowledge base to generate comprehensive and relevant lists of banking-specific keywords for both technical issues (ai_code_patch_keywords) and common customer service scenarios (vibe_workflow_keywords), as well as terms indicating ambiguity for manual review.
Logic Flow & Prioritization Design: Collaborated with the AI to refine the hierarchical decision-making process, ensuring the correct prioritization (e.g., AI Code Patch for critical technical issues taking precedence) and robust handling of various severity levels and edge cases.
Actionable Content Generation: Prompted the AI to craft practical, step-by-step checklists for "next actions" specific to each classification, and to dynamically generate detailed reasoning strings by incorporating matched keywords.
Code Quality & Documentation: Guided the AI in producing comprehensive docstrings for the class and its methods, enhancing code readability, maintainability, and adherence to Python best practices.
This collaborative approach using AI as an intelligent co-pilot allowed for efficient development, enabling me to focus on strategic design, validation, and overall solution architecture.

4. Validation Strategy Before Shipping
To ensure the reliability, accuracy, and operational readiness of this rule-based ticket classifier before its deployment, I recommend a robust, multi-pronged validation strategy:
Unit & Edge Case Testing:
Objective: Verify that the code logic functions correctly across all expected scenarios and gracefully handles invalid inputs.
Method: Develop comprehensive unit tests that cover:
All primary classification paths ("AI Code Patch", "Vibe-coded Troubleshooting Script", "Manual Review Required") with varied, representative inputs.
Edge cases such as invalid JSON input, empty or very short ticket summary fields, missing severity or channel fields, and tickets containing keywords from multiple classification categories (to verify prioritization).
Confirmation of case-insensitivity for keyword and severity matching.
Validation of the output dictionary structure.
Historical Data Simulation (Quantitative Assessment):
Objective: Quantitatively measure the classifier's performance against real-world data and identify systematic misclassification patterns.
Method: Obtain a significant dataset of actual, historically classified banking support tickets, where the ground truth (human-assigned resolution or category) is known. Process this dataset through the TicketClassifier and compare its automated output against the historical classifications.
Metrics: Calculate overall accuracy, and class-specific metrics such as Precision (how many tickets identified as X were actually X) and Recall (how many true X tickets were correctly identified).
Action: Use the results (especially insights from a confusion matrix) to iteratively refine the self.rules (add/remove keywords, adjust severity thresholds) and improve the classification logic.
Human-in-the-Loop Review (Qualitative Assessment & Rule Refinement):
Objective: Gather qualitative feedback from domain experts to validate the appropriateness of decisions, clarity of reasoning, and practical utility of action plans.
Method: Present a selected sample of the classifier's outputs (particularly for "Manual Review Required" cases, borderline classifications, or those highlighted by quantitative testing) to Subject Matter Experts (SMEs), such as senior support agents, team leads, or developers.
Feedback Focus: Collect detailed feedback on whether the decision is correct, if the reasoning is clear and logical, and if the next_actions are truly helpful and comprehensive.
Action: Conduct workshops with SMEs to collaboratively review classifier performance and refine the rule set based on their invaluable operational knowledge and evolving business needs.
Shadow Mode Deployment (Real-World, No-Risk Testing):
Objective: Observe the classifier's performance against live, real-time incoming tickets in a production-like environment without impacting actual support operations.
Method: Deploy the service in a "shadow mode" where it receives and processes live incoming support tickets. Its classification outputs are logged and monitored, but do not automatically trigger any actions or influence the current human-driven workflow.
Analysis: Continuously compare the automated classifications with how human agents actually process and resolve those same tickets. This provides invaluable real-time performance data and highlights any discrepancies or areas for final adjustments before full integration.
This comprehensive and iterative validation strategy ensures that the TicketClassifier is not only functionally correct but also accurate, robust, and trusted by the FibeAI teams who will ultimately use it.

5. Trade-offs & Future Notes
This solution, while effective and well-suited for a challenge, embodies certain design trade-offs and opens avenues for future enhancements:
Trade-offs:
Simplicity vs. Sophistication: The current rule-based system prioritizes immediate clarity, interpretability, and rapid deployment. This is excellent for auditability in a banking context. However, it may lack the nuanced understanding and adaptability of more sophisticated Machine Learning (ML) or Large Language Model (LLM) approaches when encountering highly ambiguous or entirely novel ticket summaries.
Rule-based System Characteristics:
Pros (Transparency & Control): Offers direct control over decision logic; every classification can be explicitly traced back to a specific rule, which is invaluable for debugging and compliance.
Cons (Maintenance & Rigidity): Requires manual upkeep of the self.rules dictionary. As ticket types evolve or language shifts, rules must be updated, potentially leading to increased maintenance overhead. It can also be rigid, struggling with synonyms or contextual understanding not explicitly covered by keywords.
Hardcoded vs. Externalized Rules: The classification rules are currently embedded within the Python code. While convenient for a standalone script, this design necessitates code changes and redeployment for any rule modifications.
Ambiguity and Confidence: A rule-based system provides a binary "match/no-match" decision, rather than a probabilistic "confidence score" common in ML models. While the "Manual Review Required" category serves as a critical safety net, a lack of inherent confidence metrics can make prioritization within the "Manual Review" queue less informed.
Future Notes & Potential Enhancements:
Rule Externalization: For production, the self.rules should be moved to an external configuration file (e.g., JSON, YAML), a database, or a dedicated rules engine. This enables business users (e.g., support managers) to update classification rules dynamically without requiring developer intervention or code redeployments.
Hybrid ML Integration: For tickets that fall into "Manual Review Required" due to ambiguity, a hybrid approach could be adopted. A lightweight ML model (e.g., a simple text classifier) could be trained on historical manual review cases to provide a secondary, more nuanced classification or a confidence score.
Advanced LLM Integration: Leveraging more powerful LLMs (via API) could significantly enhance capabilities:
Deeper Semantic Understanding: Better interpretation of complex or vaguely worded ticket summaries.
Dynamic Reasoning & Suggestions: Generating more sophisticated, context-aware reasoning and even suggesting potential Vibe script parameters or preliminary code investigation steps.
Novel Issue Detection: Potentially identifying entirely new categories of issues not covered by existing rules.
Automated Feedback Loop: Implement a mechanism where support agents can provide feedback on the classifier's decisions (e.g., "correct", "incorrect", "better category X"). This feedback data could then be used to continuously refine the rule set or to train and update an ML component in a hybrid system.
API Wrapper: For production deployment, this Python class would typically be wrapped within a web framework (e.g., FastAPI, Flask) to expose it as a RESTful API, allowing other services to easily integrate with it.