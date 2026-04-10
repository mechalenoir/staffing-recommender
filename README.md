# Staffing Recommender Agent

An AI-powered staffing optimization agent built for SAP consulting and ERP implementation companies. The agent proactively suggests the optimal consulting team for a sales opportunity during the Negotiation stage in Salesforce — minimizing cost while ensuring quality delivery.

---

## The Problem

When a sales opportunity enters the Negotiation stage, a pre-sales or delivery manager must manually browse through a headcount tool, evaluate consultant availability, match skills to project requirements, and estimate costs — all while juggling multiple active projects and ongoing proposals. This process is slow, inconsistent, and often results in either over-staffing (high cost) or under-staffing (quality risk).

---

## The Solution

The Staffing Recommender agent automates the team suggestion process. Given a project opportunity, it:

1. Reads the project requirements (modules, timeline, region, client)
2. Queries the consultant roster for available, skill-matched candidates
3. Checks for conflicts with active projects
4. Applies the correct rate logic (specific rate → MSA rate → seniority/region tier)
5. Optimizes team composition to minimize cost while meeting quality thresholds
6. Presents a full recommended team with cost estimate — for human review and approval

The agent never books or assigns anyone. A human always approves the final team.

---

## Salesforce Stage Integration

| Stage | Agent Action |
|---|---|
| Qualification → Proposal | No action |
| **Negotiation** | ✅ Generates initial team recommendation + cost estimate for SOW |
| Statement of Work | Monitors for renegotiation triggers |
| **Negotiation (re-entry)** | ✅ Re-runs recommendation with updated constraints |
| **Closed** | ✅ Confirms final team for handoff to staffing/headcount tool |

---

## Tech Stack

- **Claude AI** — core reasoning engine (knowledge retrieval, team optimization logic, cost calculation)
- **RAG / Knowledge Base** — document store containing consultant roster, rate tiers, MSA rates, and active project data
- **Salesforce** — source system for opportunity data and stage triggers
- **Excel / XLSX** — structured data format for consultant and rate knowledge base

---

## Knowledge Base Documents

The agent's knowledge base contains two documents:

### 1. `SAP_Staffing_Knowledge_Base.xlsx`
Four sheets:
- **Consultant Roster** — 100 SAP consultants with name, seniority, region, skills, certifications, daily rate (USD), availability %, and current status
- **Rate Tiers by Seniority** — fallback rate ranges by seniority level and region (used when consultant is not yet named)
- **MSA Rates** — pre-negotiated rates per client and seniority level (takes priority over standard tiers)
- **Active Projects** — current project assignments used for conflict detection

### 2. `Agent_Instructions_and_Logic.txt`
The agent's reasoning guide covering:
- Team composition templates by project size
- Rate priority logic (specific → MSA → tier)
- Conflict detection rules
- Output format specification

---

## How to Use

Start a conversation with the agent and describe your opportunity:

**Example prompt:**
> "I have an opportunity entering Negotiation. Client is Acme Manufacturing, SAP S/4HANA migration, modules FI, CO and MM, 6 months, North America delivery. Suggest a team."

The agent will return:
- Recommended team table (name, seniority, module, allocation %, daily rate, estimated cost)
- Total project cost estimate
- Rate basis used (specific / MSA / tier)
- Conflict flags for any partially allocated consultants
- Alternative options if cost optimization is needed

---

## Rate Priority Logic

```
1. Named consultant specific rate  →  most accurate
2. Client MSA agreed rate          →  pre-negotiated, takes priority over tiers
3. Seniority + region tier rate    →  ballpark estimate when consultant not yet named
```

---

## Team Composition Model

| Role | Seniority | Notes |
|---|---|---|
| Project Lead | Senior Manager / Principal | Owns delivery accountability |
| Functional Lead | Manager / Senior Consultant | One per major SAP module |
| Consultant | Consultant / Senior Consultant | Configuration and testing |
| Analyst | Analyst / Consultant | Data migration, documentation, testing |

Team size scales with project complexity:
- Small (< 3 months): 3–5 people
- Mid-size (3–6 months): 5–10 people
- Large (6–12 months): 10–20 people

---

## Roadmap (v2)

The current version is a single-agent MVP. The production architecture will evolve into a multi-agent system:

- **Salesforce Listener Agent** — monitors stage changes and triggers the workflow automatically
- **Staffing Recommender Agent** — core optimization logic (current agent)
- **Handoff Agent** — validates approved team against the approved SOW and pushes confirmed team into the staffing tool
- **Coordinator Agent** — orchestrates all three and passes context between them
