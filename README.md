# 2026-Sentinel-Core-A-Multi-Layered-Cyber-Resilient-Framework-for-Securing-Autonomous-LLM-Assistants
https://idea.unisys.com/D8958

### **Description**
Sentinel-Core is a cyber-resilient framework designed to enhance the security of autonomous AI systems operating in dynamic and high-risk environments. The framework adopts a structured, multi-layered approach to ensure that AI-driven decisions remain aligned with intended user goals.
By combining intent-aware validation, controlled reasoning flow, and policy-driven execution checks, the system minimizes the risk of unintended or unsafe actions. It further strengthens reliability through isolated execution environments that restrict unauthorized operations.
Overall, Sentinel-Core shifts AI security toward a more robust and adaptive architecture, enabling trustworthy deployment of intelligent agents in enterprise applications.

The system implements comprehensive security controls across multiple functional layers:

- **Perception Layer** - Input analysis and threat detection
- **Governance Layer** - Decision-making and action validation
- **Risk Engine** - Hybrid risk analysis
- **Isolation Layer** - Security isolation mechanisms
- **Planning Layer** - Safe task planning
- **Executor Layer** - Controlled task execution
- **Judge Layer** - Output validation and safety verification

The framework includes REST API endpoints for analyzing email/text inputs and executing tasks while maintaining security constraints.

### **Setup**

**Prerequisites:**
- Python 3.x
- Flask web framework
- Ollama - qwen 2.5 13B

**Installation Steps:**

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Dependencies:
   - Flask
   - flask-cors (for CORS support)
   - ollama (LLM backend)

3. Run the application:
```bash
python app.py
```

**API Endpoints:**
- `GET /` - Home route with API information
- `POST /analyze` - Analyze email/text inputs through the security pipeline
- `POST /task` - Execute secure tasks
- `GET /demo` - Demo endpoint

**Project Structure:**
- app.py - Main Flask application
- layers - Security layer implementations (risk_engine, governance, isolation, planner, executor, judge, perception)
- data - Training and test datasets
- extension - Browser extension files
- convert_dataset.py - Dataset conversion utilities
- evaluate.py - Evaluation scripts

## **Detailed Code Architecture**

### **1. Main Application (app.py)**

**Core Flask Routes:**
- **`GET /`** - Health check endpoint returning API info
- **`POST /analyze`** - Email analysis pipeline: Perception → Governance
- **`POST /task`** - Full task execution: Perception → Governance → Planner → Executor → Judge

**Analyze Flow:**
```python
# Step 1: Perception (Risk Analysis)
result = analyze_hybrid(text)
trust_score = result["trust_score"]

# Step 2: Governance (Decision)
status = decide(trust_score)

# Returns: status, trust_score, types, reason, flagged_count, flagged_lines
```

**Task Flow:**
```python
# Step 1: PERCEPTION - Analyze email security
security_result = analyze_hybrid(email_text)

# Step 2: GOVERNANCE - Block if malicious
if decide(trust_score) == "BLOCKED":
    return "Blocked mail cannot be processed"

# Step 3: PLANNER - Check if task is allowed
plan_result = plan(command)  # Only allows: summarize, simplify, reply, action items

# Step 4: EXECUTOR - Run approved task
# Step 5: JUDGE - Validate output
# Step 6: RETURN - Safe results
```

---

### **2. Perception Layer** (perception.py)

**Key Functions:**

**`normalize_text(text)`** - Preprocesses input
- Joins spaced characters (i g n o r e → ignore)
- Converts to lowercase
- Removes punctuation noise
- Normalizes whitespace

**`try_decode_base64(blob)`** - Decodes base64 payloads
- Handles URL-safe Base64 (replaces - and _ with + and /)
- Fixes missing padding
- Filters meaningful payloads (>8 chars)
- Used to detect obfuscated malicious content

**`decode_base64_payloads(text)`** - Extracts and decodes Base64 blocks from emails

**`analyze_email(email_text)`** - Main perception analysis
- Uses `detect_intent()` for intent classification
- Extracts and decodes Base64 payloads
- Detects phishing patterns (urgency keywords, verification requests)
- Identifies prompt injection attempts
- Returns: `trust_score`, `malicious`, `types`, `flagged_lines`, `reason`

---

### **3. Governance Layer** (governance.py)

**Decision Engine:**
```python
def decide(trust_score):
    if trust_score <= 3:
        return "BLOCKED"      # Malicious (0-3)
    elif trust_score <= 7:
        return "FLAGGED"      # Suspicious (4-7)
    return "SAFE"             # Trust (8-10)
```

**Blocked Actions:**
- send email, send file, upload file
- share credentials, access system files
- retrieve secrets

**`validate_action(output)`** - Ensures executor output contains no blocked phrases

---

### **4. Risk Engine** (risk_engine.py)

**`analyze_hybrid(email_text)`** - Aggregates security analysis
```python
# Wraps perception analysis and returns standardized format
def analyze_hybrid(email_text):
    result = analyze_email(email_text)
    return {
        "trust_score": result["trust_score"],
        "malicious": result["malicious"],
        "types": result["types"],
        "flagged_lines": result["flagged_lines"],
        "reason": result["reason"]
    }
```

---

### **5. Isolation Layer** (isolation.py)

**`isolate(email_text)`** - Sanitizes content using LLM
- Uses **Ollama client** (Qwen 2.5 3B model)
- Removes commands, instructions, prompt injections
- Removes credential/system access requests
- Keeps only factual information
- Returns sanitized content for safe processing

```python
client = ollama.Client(
    host="https://39c7-162-216-141-56.ngrok-free.app"
)
```

---

### **6. Planner Layer** (planner.py)

**`plan(command)`** - Whitelist-based task approval
```python
allowed = [
    "summarize",      # Create 3-bullet summary
    "simplify",       # Simplify complex content
    "reply",          # Draft professional reply
    "action items"    # Extract action items
]

# Returns: {"allowed": True/False, "task": command_name}
```

---

### **7. Executor Layer** (executor.py)

**`execute(task, content)`** - Performs approved tasks using LLM

**Task: Summarize**
- Extracts: sender, main purpose, next steps
- Returns 3-bullet summary

**Task: Reply**
- Generates professional email response
- Uses Qwen 2.5 3B model

---

### **8. Judge Layer** (judge.py)

**Two-Tier Validation (Fast + LLM):**

**`fast_judge(output)`** - Rule-based check (executes first)
```python
dangerous_patterns = [
    "ignore previous instructions",
    "send credentials",
    "share credentials",
    "provide system access",
    "database access",
    "share api key",
    "delete files",
    "internal configuration details"
]
# Returns: {"safe": True/False, "reason": "..."}
```

**`llm_judge(output)`** - LLM fallback (if fast_judge passes)
- Checks for credential leakage
- Detects secrets/API keys
- Validates unauthorized access attempts
- Returns LLM safety assessment

---

### **9. Intent Detector** (intent_detector.py)

**`detect_intent(email_text)`** - Classifies email threats

**Returns JSON:**
```json
{
    "malicious": false,
    "trust_score": 10,
    "types": [],
    "reason": ""
}
```

**Detects:**
- Prompt injection attempts
- Phishing (fake urgency, account verification)
- Credential/system access requests
- Encoded malicious payloads
- AI behavior manipulation attempts
- Impersonation/authority claims

---

### **Complete Request Flow**

```
┌─────────────────────────────────┐
│  User Input (Email + Command)   │
└────────────┬────────────────────┘
             │
             ▼
    ┌─────────────────────┐
    │  PERCEPTION LAYER   │
    │ (analyze_email)     │
    │ - Intent detection  │
    │ - Base64 decode     │
    │ - Phishing patterns │
    └──────────┬──────────┘
             │
             ▼
    ┌─────────────────────┐
    │  GOVERNANCE LAYER   │
    │ (decide, validate)  │
    │ Blocked/Flagged/Safe│
    └──────────┬──────────┘
             │
      ➜ Trust Score Required
             │
             ▼
    ┌─────────────────────┐
    │  PLANNER LAYER      │
    │ (plan)              │
    │ Whitelist commands  │
    └──────────┬──────────┘
             │
      ➜ Task Approval
             │
             ├─ Fast Task ─────────────────┐
             │                             │
             ▼                             ▼
    ┌─────────────────────┐      ┌─────────────────────┐
    │  EXECUTOR LAYER     │      │  ISOLATION LAYER    │
    │ (execute)           │      │ (isolate)           │
    │ - Summarize         │      │ Neutralize content  │
    │ - Reply             │      │ then Execute        │
    │ - Action items      │      │                     │
    └──────────┬──────────┘      └──────────┬──────────┘
             │                             │
             └──────────────┬──────────────┘
                            │
                            ▼
                  ┌─────────────────────┐
                  │   JUDGE LAYER       │
                  │ (fast_judge → llm)  │
                  │ - Rule-based check  │
                  │ - LLM validation    │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  Return Safe Result │
                  └─────────────────────┘
```
---

### **Key Security Features**

1. **Multi-layered validation** - Content passes 7+ security checkpoints
2. **Intent detection** - Identifies prompt injections, phishing, malicious directives
3. **Entropy analysis** - Detects obfuscated payloads (Base64, spaced characters)
4. **Content isolation** - Strips dangerous instructions before processing
5. **Output validation** - Ensures executor doesn't leak secrets/credentials
6. **LLM-based judgment** - Uses Qwen model for semantic safety checks
7. **Trust scoring** - 0-10 scale (0-3 dangerous, 4-7 suspicious, 8-10 safe)

---

### **Technology Stack**
- **Framework:** Flask + Flask-CORS
- **LLM Backend:** Ollama (Qwen 2.5 3B model)
- **Remote Host:** ngrok tunnel (https://39c7-162-216-141-56.ngrok-free.app)
- **Language:** Python 3
