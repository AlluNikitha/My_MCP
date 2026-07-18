# AURA — Smart Campus Operations Agent

AURA (Automated Unified Reasoning Agent) is a next-generation Smart Campus Operations Agent built for the Model Context Protocol (MCP) hackathon. 

Rather than presenting students with a dozens of disconnected CRUD forms and dashboards, AURA consolidates campus services into **9 domain-specific MCP tool servers** and uses an **Agent Orchestrator (powered by Claude / Gemini or a smart Local Agent Executor)** to chain operations dynamically in a single natural language request.

![Architecture Diagram](docs/architecture.png)

---

## Key Features

1. **Agent-First Operations:** The centerpiece is the `AgentConsole` chat window, which accepts natural language queries, reasons about them, invokes appropriate MCP tool pipelines, and displays a step-by-step execution trace.
2. **Write/Action Capability:** AURA is not just a search tool. Every server contains write capabilities (filing outpasses, marking grievances resolved, making tuition payments, and submitting job registrations).
3. **Decoupled Architecture:** Each of the 9 domain MCP servers operates independently (scoped only to its database entity) and can be run standalone. All cross-domain composition logic is isolated in the backend orchestrator.
4. **Offline Resilience:** If no Anthropic API key is provided, AURA falls back to a smart Local Agent Executor which still resolves the identical multi-tool operations pipelines against the real database files, ensuring a robust out-of-the-box demo.

---

## File Structure

```
AURA/
├── frontend/
│   ├── src/
│   │   ├── pages/         # Dashboard, Attendance, Hostel, Finance, Placements, etc.
│   │   ├── components/    # Sidebar, Navbar, cards, and AgentConsole.jsx
│   │   └── services/      # api.js API client
├── backend/
│   ├── api/               # FastAPI route mappings for system-of-record views
│   ├── agent/             # orchestrator.py (MCP stdio client session coordinator)
│   └── main.py            # FastAPI main server
├── mcp_servers/           # The 9 domain-specific standalone FastMCP servers
├── data/                  # Thread-safe JSON databases (students, attendance, timetable, etc.)
└── docs/                  # Pitch deck, architecture diagrams, and workflow charts
```

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- Node.js (v18+)

### Backend Installation
1. Install Python packages:
   ```bash
   pip install -r requirements.txt
   ```
2. (Optional) Provide your Anthropic API Key in `.env`:
   ```env
   # .env
   ANTHROPIC_API_KEY=your_key_here
   ```
   *If left empty, AURA dynamically uses its smart Local Agent Executor fallback, executing identical real-tool runs.*

### Frontend Installation
1. Go to the frontend directory:
   ```bash
   cd frontend
   ```
2. Install package dependencies:
   ```bash
   npm install
   ```

---

## Running AURA

To demonstrate the full experience, we will run the backend, the frontend, and verify the standalone MCP servers.

### 1. Start the FastAPI Backend
Start the backend from the root directory:
```bash
python -m uvicorn backend.main:app --port 8000 --reload
```
*At startup, the backend automatically starts all 9 MCP servers as subprocesses using standard input/output transport client sessions.*

### 2. Start the React Frontend
Start the Vite development server from the `frontend/` folder:
```bash
npm run dev
```
Open `http://localhost:3000` in your web browser.

### 3. Running MCP Servers Standalone
Any server can be run independently and connected to external desktop MCP clients (like Claude Desktop) for live testing:
```bash
python mcp_servers/attendance_mcp.py
python mcp_servers/hostel_mcp.py
```

---

## Demo walkthrough Script (2–3 Minutes)

Use AURA's top-right **Student Context Switcher** to demonstrate different outcomes based on student stats:
- **Arjun Sharma (S1001):** High GPA (8.2), zero backlogs, paid tuition fee, 85% average attendance.
- **Sneha Reddy (S1002):** GPA (7.4), 1 backlog, ₹15,000 tuition unpaid, 77% average attendance (with low attendance in EC302).

---

### Scenario 1: Placement Eligibility Check (Flow A)
- **Concept:** Chains 4 servers (`attendance`, `finance`, `library`, `placement`) in a single query.
- **Demo steps:**
  1. Select **Arjun Sharma** context.
  2. Click **Flow A — Placement Check** query in the Agent Console.
  3. Notice the agent invokes:
     - `attendance.get_attendance_percentage` (returns 80.5%)
     - `finance.get_finance_dues` (returns 0 dues)
     - `library.get_library_dues` (returns 0 dues)
     - `placement.get_placement_profile` (returns CGPA 8.2, 0 backlogs)
  4. Arjun is declared **ELIGIBLE**.
  5. Switch active student context to **Sneha Reddy**.
  6. Click **Flow A — Placement Check** again.
  7. Notice that Sneha is declared **INELIGIBLE** due to outstanding finance dues and academic backlogs.

---

### Scenario 2: Weekend Outpass approval (Flow B)
- **Concept:** Chains 3 servers (`hostel`, `attendance`, `complaint`) + executes a write transaction (`hostel.file_outpass_request`).
- **Demo steps:**
  1. Select **Arjun Sharma** context.
  2. Click **Flow B — Hostel Outpass** query.
  3. Notice the agent checks leave policy, verifies Arjun's attendance is above 75%, and scans for any open hostel complaints.
  4. Seeing Arjun is eligible, it calls the write tool `hostel.file_outpass_request` to register the outpass request.
  5. Switch to **Hostel** page on the sidebar. Arjun's leave list now lists the new approved outpass log.

---

### Scenario 3: Timetable watchdog (Flow C)
- **Concept:** Proactively identifies schedule conflicts.
- **Demo steps:**
  1. Click **Flow C — Conflict Watchdog** query.
  2. The watchdog queries Arjun's timetable and registered events, detects that a Monday tutorial clashes with the registered *Google Hackathon Prep Meet*, and calls `attendance.flag_attendance_shortage` to file an academic warning/reconciliation request automatically.
  3. Navigate to **Complaints** on the sidebar to inspect the new filed grievance.
