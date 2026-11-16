# Agent recipes — Client CRM + Deal Pipeline (base a)

Natural-language prompts for an **MCP-capable agent** connected to this Airtable base (read tools for briefings; write tools only when you explicitly allow updates).

Harlo is **not affiliated with Airtable**. These recipes describe how an agent can operate on a base that **works with Airtable**.

Public support for this kit: **digitalpromohub.support@gmail.com**

**Rules for the agent (paste once as standing instructions):**

- Treat all currency fields labeled `(estimate)` as **estimates**, never as booked revenue.  
- Do not invent contacts, emails, or companies. If a record is missing, say so.  
- Prefer read-only analysis unless the user says “apply,” “update,” or “fix.”  
- Never message customers directly. Drafts only.  
- Respect `Do Not Contact`.  
- Sample CSV people and `.example` domains are fictional — say so if they still appear in the base.  
- Do not claim affiliation with Airtable.

---

## Recipe A — Daily briefing

**When:** Morning, or on demand (“brief me”).

**Prompt:**

```
You are assisting with the Harlo Client CRM + Deal Pipeline base that works with Airtable.

Produce a daily briefing for today in America/Denver local time. Use only live base data.

Sections:
1) Pipeline snapshot — count of open deals (Stage not Won/Lost); sum of Amount (estimate); sum of Weighted Amount (estimate). Label every dollar figure as an estimate.
2) Due today — Activities with Status Planned and Date = today; Deals with Next Step Due = today.
3) Overdue — Activities Overdue; Deals with Next Step Due before today still open.
4) Stale deals — Stale? = Stale; list Deal Name, Owner, Stage, Days in Stage, Amount (estimate), Next Step.
5) Closing in the next 14 days — open deals by Close Date ascending.
6) Risks — Blocker contacts on open deals; Companies with Health Red linked to open deals.
7) Suggested focus (max 5 bullets) — concrete next actions for owners; no fluff.

Formatting: tight Markdown. No logos. End with: “Figures are estimates. Harlo is not affiliated with Airtable.”
If MCP tools error, report which tool failed and stop; do not fabricate rows.
```

**Optional follow-up:**

```
Turn section 7 into a checklist grouped by Owner name from Team Members.
```

---

## Recipe B — Data hygiene

**When:** Weekly, or before a forecast meeting.

**Prompt:**

```
You are running data hygiene on the Harlo Client CRM + Deal Pipeline base (works with Airtable; Harlo is not affiliated with Airtable).

Audit and report — do not write yet:

1) Contacts with Data Quality ≠ Complete, or missing Email, or missing Company, or Hygiene flag Needs attention.
2) Contacts with Do Not Contact checked that still appear as Primary Contact on open Deals — flag conflict.
3) Companies with Status Prospect but Won Deal Count > 0 (or linked Won deals) — status drift.
4) Companies / Deals / Contacts with empty Owner while Active team exists.
5) Deals missing Close Date, Amount (estimate), Primary Contact, or Next Step while Stage not Won/Lost.
6) Deals in Lost with empty Lost Reason.
7) Activities Planned with Date more than 7 days ago.
8) Duplicate risk: same Contact Email or same Company Domain appearing more than once.

Output:
- A severity-ordered table (Critical / High / Medium) with record name, table, issue, suggested fix.
- A “safe auto-fix list” (only unambiguous fixes, e.g. set Data Quality = Needs email when Email blank).
- A “needs human” list.

Wait for me to say “apply safe fixes” before using any write tools.
```

**Apply prompt (only after review):**

```
Apply only the safe auto-fix list from your last hygiene report. Summarize each write (table, record, field, old → new). Skip anything that touches Do Not Contact, Stage, Amount (estimate), or email sends.
```

---

## Recipe C — Pipeline review

**When:** Weekly forecast / 1:1s.

**Prompt:**

```
Run a pipeline review on the Harlo Client CRM + Deal Pipeline base. Works with Airtable; not affiliated with Airtable.

Scope: open deals only (Stage not Won/Lost).

For each deal provide:
- Deal Name, Company, Owner, Stage, Amount (estimate), Probability %, Weighted Amount (estimate), Close Date, Forecast Category, Days in Stage, Stale?, Next Step, Next Step Due
- Primary Contact + Role Type
- Last 3 Activities by Date (Subject, Type, Status, Outcome snippet)

Then roll up:
1) By Stage — count, Amount (estimate) sum, Weighted sum
2) By Owner — same
3) By Forecast Category — same
4) Commit reliability notes — Commit deals missing Next Step Due or with Days in Stage > 14
5) Slip risk — Close Date in the past but not Won/Lost
6) Top 5 deals by Weighted Amount (estimate) — one-line coaching question each

All money = estimates. End with three questions I should ask in the forecast meeting.
Do not change Stage or Amount unless I ask.
```

**Deep-dive prompt:**

```
Deep-dive the deal named “{DEAL_NAME}”. Summarize company context, buying roles, open risks (Blocker, Red health, overdue activities), and draft a short next-step email to the Champion — draft only, do not send. Keep tone professional and brief. Reminder: amounts are estimates.
```

---

## Recipe D — Standing “tools” cheatsheet for the agent

If your MCP server exposes Airtable-style tools, map them like this (names vary by connector):

| Intent | Typical tool pattern |
|--------|----------------------|
| List / filter rows | `list_records` / `search` on table name |
| Read one row | `get_record` |
| Hygiene fixes | `update_record` on allowed fields only |
| Log a note | `create_record` on Activities |

Standing deny list unless user overrides: sending email, deleting records, bulk Stage changes, exporting the full base to a public URL.

---

## Recipe E — One-liner triggers (chat shortcuts)

| User says | Run |
|-----------|-----|
| `briefing` | Recipe A |
| `hygiene` | Recipe B |
| `pipeline` | Recipe C |
| `stale` | Recipe A sections 3–4 only |
| `forecast` | Recipe C rollups 1–3 only |

---

## Sample first message to lock context

```
Use the Harlo Client CRM + Deal Pipeline schema (Companies, Contacts, Deals, Activities, Team Members). Currency fields are estimates. Do not contact anyone. Do not publish. Support contact for the kit itself is digitalpromohub.support@gmail.com. Harlo is not affiliated with Airtable. Ready for “briefing”, “hygiene”, or “pipeline”.
```
