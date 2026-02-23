# AI Employee - Silver Tier Specification

**Version:** 2.0.0 | **Date:** 2026-02-15 | **Tier:** Silver | **Prerequisites:** Bronze Complete

---

## Overview

Silver Tier adds approval workflows, WhatsApp/LinkedIn integration, AI planning, and 7 automated workflows to Bronze.

### Bronze → Silver Evolution

| Aspect | Bronze | Silver |
|--------|--------|--------|
| Sources | Gmail, Filesystem | + WhatsApp, LinkedIn |
| Processing | Auto-categorize | + Approval workflows |
| Planning | None | AI-generated plans |
| Dashboard | Simple counts | Real-time multi-platform |
| Storage | Files only | + SQLite database |
| Reports | None | Weekly/monthly automated |

---

## Test-Driven Development (Mandatory)

**TDD Cycle:** RED (write failing test) → GREEN (make it pass) → REFACTOR → DOCUMENT → COMMIT

**Coverage:** Minimum 90% for all Silver code.

**Must Test:** Approval logic, database operations, workflows, financial calculations, error handling, edge cases.

---

## System Architecture

```
┌─────────────────────────────────────────┐
│  Data Sources: Gmail | Files | WhatsApp | LinkedIn  │
└────────────────────┬────────────────────┘
                     ↓
┌─────────────────────────────────────────┐
│  Unified Inbox → Intelligent Processing │
│  (categorize, prioritize, value check)  │
└────────────────────┬────────────────────┘
                     ↓
        ┌────────────┴────────────┐
   HIGH VALUE?                NORMAL?
        ↓                          ↓
┌───────────────┐          ┌─────────────┐
│Pending_Approval│          │Needs_Action │
│  (user decides)│          │(auto-process)│
└───────┬───────┘          └──────┬──────┘
        └────────────┬────────────┘
                     ↓
        ┌─────────────────────────┐
        │   Workflow Orchestrator  │
        │  (7 automated workflows) │
        └────────────┬────────────┘
                     ↓
        ┌─────────────────────────┐
        │  Vault + SQLite Database │
        └─────────────────────────┘
```

---

## Vault Structure

```
AI_Employee_Vault/
├── .claude/skills/           # 12 skills total (4 Bronze + 8 Silver)
│   ├── vault-management, email-processor, file-organizer, dashboard-updater
│   └── approval-manager, plan-generator, workflow-orchestrator
│   └── whatsapp-processor, linkedin-processor, financial-tracker
│   └── enhanced-dashboard, report-generator
├── Dashboard.md              # Real-time multi-platform view
├── Company_Handbook.md       # Business rules & configuration
├── Inbox/                    # Raw incoming items
│   ├── emails/, files/, whatsapp/, linkedin/
├── Needs_Action/             # Auto-processed items (<$1000)
│   ├── urgent/, normal/
├── Pending_Approval/         # High-value awaiting decision (NEW)
│   ├── high_value/, workflows/
├── Approved/                 # Approved + workflows executed (NEW)
├── Rejected/                 # Rejected with reasons logged (NEW)
├── Plans/                    # AI-generated execution plans (NEW)
│   ├── active/, pending_approval/, completed/
├── Done/                     # Completed items
├── Logs/                     # Audit trail
│   ├── daily/, approvals/, workflows/
├── Reports/                  # Generated reports (NEW)
│   ├── weekly/, monthly/, custom/
└── Database/                 # SQLite storage (NEW)
    └── ai_employee.db
```

---

## Database Schema

**Tables:** `items`, `approvals`, `plans`, `workflows`, `financial_records`, `activity_log`

**Operations:** All DB access through centralized utility with connection pooling, transactions, error handling.

---

## Features

### 1. Approval Workflow

**Triggers:** Invoice ≥$1000, contracts, "approval required" keyword

**Config:** `approval_threshold` ($1000), `approval_timeout_hours` (24), `approval_reminder_hours` (4), `auto_approve_on_timeout` (true)

**Flow:**
1. Item processed by skill
2. approval-manager checks value against threshold
3. Route to Pending_Approval/, generate approval card
4. User approves/rejects via Obsidian or CLI
5. Approved → move to Approved/, execute workflow; Rejected → move to Rejected/, log reason; Timeout → auto-approve after 24h

**Approval Card:** Header "APPROVAL REQUIRED", item details, financial impact, AI recommendation, decision instructions, deadline.

**Tests:** Threshold detection, contract detection, card generation, approval/rejection/timeout logic.

### 2. AI Planning System

**Triggers:** Keywords "plan", "organize", "prepare", "research", "coordinate"; multi-step tasks (>$500, >3 steps)

**Types:**
- **Simple:** Checklist, <2 hours, 5-10 steps
- **Detailed:** Phase-based, >2 hours, timelines, budgets, approval points

**Storage:** Plans/active/, Plans/pending_approval/, Plans/completed/

**Tests:** Trigger detection, plan type selection, generation, progress tracking, state management.

### 3. WhatsApp Integration

**Included:** Direct messages with urgent keywords, group mentions, important contacts, media

**Excluded:** Casual chats, group chatter without mention, promos, status updates

**Watcher:** Node.js + whatsapp-web.js, QR auth, real-time monitoring

**Output:** `YYYY-MM-DD-HHMM_whatsapp_SenderName.md` with sender, timestamp, priority, body, attachments

**Tests:** Filtering, metadata extraction, media download, markdown generation, priority detection.

### 4. LinkedIn Integration

**Included:** Job opportunities, relevant connection requests, DMs with keywords (opportunity, position, role, project)

**Excluded:** Generic promos, irrelevant connections, feed updates, automated notifications

**Watcher:** Python + LinkedIn API, OAuth 2.0, 5-minute polling

**Output:** `YYYY-MM-DD-HHMM_linkedin_SenderName.md` with profile, company, salary, location, deadline

**Tests:** OAuth, filtering, data extraction, markdown generation, rate limiting.
*Note: LinkedIn Poster currently supports text, images, and links. Document support is planned for the next tier.*

### 5. Workflow Orchestration

**7 Workflows:**

| Workflow | Trigger | Approval | Output |
|----------|---------|----------|--------|
| Invoice Processing | Invoice detected | If ≥$1000 | Financial record, reminder |
| Receipt Processing | Receipt detected | None | Expense record, report update |
| Research | Keywords: research/find/compare | None | Research report |
| File Organization | New file detected | None | Organized file, metadata |
| Email Response | Important email | Always | Sent email, log |
| Meeting Preparation | Meeting invite | None | Agenda, reminders, tasks |
| Expense Report | End of month | If >$500 | Report (PDF+CSV) |

**State Management:** Tracked in database (type, step, status, timestamps, state JSON)

**Capabilities:** Pause (at approval/errors), Resume, Rollback, Retry (3 attempts)

**Tests:** End-to-end workflows, approval pausing, resume, error handling, state persistence, concurrency.

### 6. Financial Tracking

**Tracked:** Invoices (amount, vendor, due date, status), Expenses (amount, payee, date, category, receipt)

**Categories:** Defined in Company_Handbook.md (Meals, Travel, Supplies, Software, Services, Marketing...)

**Budgets:** Monthly total + per-category; alerts at 80%, critical at 95%

**Dashboard Shows:** Pending/paid invoices, expenses, budget %, overdue alerts

**Tests:** Amount extraction, categorization, budget calculations, overdue detection, report generation, data accuracy.

### 7. Enhanced Dashboard

**Sections:**
- Multi-platform summary table (4 sources)
- Pending approvals (priority, amount, deadline)
- Active plans (progress %)
- Workflow status (step, running/paused)
- Financial tracking (pending, paid, expenses, budget)
- Recent activity (last 10 actions)
- System status (watchers, DB, backup)

**Updates:** Real-time on events, every 60s scheduled, on-demand

**Tests:** Counting, approval queue, plan progress, workflow status, financial totals, real-time updates.

### 8. Automated Reports

| Type | Schedule | Format | Location |
|------|----------|--------|----------|
| Weekly Activity | Monday 9am | Markdown | Reports/weekly/ |
| Monthly Financial | 1st 9am | Markdown+CSV | Reports/monthly/ |
| Custom | On-demand | Markdown+CSV | Reports/custom/ |

**Weekly Contains:** Items by source, top categories, approvals, plans, workflows, time saved

**Monthly Contains:** Invoices, payments, expenses by category, budget vs actual, trends, recommendations

**Tests:** Scheduled generation, data accuracy, date ranges, CSV export, formatting, file creation.

---

## Silver Skills (8 New)

| Skill | Purpose | When Used |
|-------|---------|-----------|
| approval-manager | Approval workflow | High-value items |
| plan-generator | AI execution plans | Keywords or complex tasks |
| workflow-orchestrator | Multi-step automation | Matching workflow pattern |
| whatsapp-processor | Process WhatsApp | New WhatsApp message |
| linkedin-processor | Process LinkedIn | New LinkedIn item |
| financial-tracker | Invoice/expense tracking | Financial items |
| enhanced-dashboard | Real-time dashboard | Event or scheduled update |
| report-generator | Weekly/monthly reports | Scheduled or on-demand |

---

## Configuration

**Company_Handbook.md:** Business rules (thresholds, timeouts, budgets, categories, contacts)

**.env File:** Secrets (API keys, DB path, tokens)

**Key Options:**
- `approval_threshold`: $1000
- `approval_timeout_hours`: 24
- `monthly_budget`: $5000
- `budget_alert_threshold`: 80%
- `weekly_report_day`: Monday
- `monthly_report_day`: 1

---

## Edge Cases

**Approval:** Duplicate decisions (ignore), timeout during downtime (process on restart), corrupted card (regenerate), negative amounts (flag), missing amount (require approval)

**Workflow:** Paused on restart (resume), rejected mid-flow (mark failed), concurrent workflows (DB locking), repeated failures (3 retries then fail), missing data (pause for input)

**Financial:** Duplicates (check 7-day window), budget exceeded (flag), unknown category ("Uncategorized"), multi-currency (convert to USD), zero amounts (skip)

**Database:** Locked (retry 3x), corrupted (restore backup), concurrent writes (transaction isolation), missing tables (auto-create)

---

## Implementation

### What Exists (Bronze)
- Gmail/filesystem watchers, 4 skills (vault-management, email-processor, file-organizer, dashboard-updater)
- Vault structure, basic Dashboard.md, Company_Handbook.md, test infrastructure

### What to Build (Silver)
- **Infrastructure:** SQLite DB, new folders (Pending_Approval, Approved, Rejected, Plans, Reports, Database)
- **Watchers:** WhatsApp (Node.js), LinkedIn (Python)
- **Skills:** 8 new listed above
- **Workflows:** 7 implementations

### 10 Phases (~10 days)
1. Database & Infrastructure
2. Approval Workflow
3. WhatsApp Watcher
4. LinkedIn Watcher
5. Planning System
6-7. Workflow System (2 days)
8. Financial & Reports
9. Enhanced Dashboard
10. Integration Testing

**Tests:** Unit, integration, database, end-to-end, error handling. Min 80% coverage.

---

## Success Criteria

Silver complete when:
1. All 4 watchers running
2. Approval workflow functional
3. Planning system working
4. 5+ workflows implemented
5. Database operational
6. Enhanced dashboard live
7. Financial tracking accurate
8. Reports generating on schedule
9. All tests passing (>80% coverage)
10. Documentation complete

---

## References

- **docs/project_info/PROJECT_KICKOFF_SILVER.md** - Implementation plan
- **docs/platforms/WHATSAPP_LINKEDIN_SETUP.md** - Platform setup guide
- **Company_Handbook.md** - Configuration

---

*End of Silver Tier Specification*
