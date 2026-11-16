# Schema — Client CRM + Deal Pipeline (base a)

Brand: **Harlo**. This base **works with Airtable**. Harlo is **not affiliated with Airtable**.

Fictional sample rows live in `data/`. Replace them with your own records. Any dollar amounts below are **estimates** or illustrative formula examples, not promises.

---

## Tables (5)

| # | Table | Primary field | Purpose |
|---|-------|---------------|---------|
| 1 | **Team Members** | Name (single line) | Owners who appear on deals, contacts, and activities |
| 2 | **Companies** | Company Name | Accounts / organizations |
| 3 | **Contacts** | Full Name | People at companies |
| 4 | **Deals** | Deal Name | Opportunities in the pipeline |
| 5 | **Activities** | Subject | Calls, emails, meetings, tasks |

Create tables in that order so link targets exist when you add linked-record fields.

---

## 1. Team Members

| Field | Type | Options / notes |
|-------|------|-----------------|
| **Name** | Single line text | Primary |
| **Email** | Email | Work email |
| **Role** | Single select | `AE`, `SDR`, `CSM`, `Manager`, `Ops` |
| **Active** | Checkbox | Uncheck for departed owners |
| **Deals Owned** | Link to Deals | Allow multiple; created from Deals → Owner |
| **Contacts Owned** | Link to Contacts | Allow multiple; from Contacts → Owner |
| **Activities Owned** | Link to Activities | Allow multiple; from Activities → Owner |
| **Open Deal Count** | Count | Count of Deals Owned where Stage is not Won/Lost (via view or Rollup — see formulas) |
| **Notes** | Long text | Internal |

### Views — Team Members

- **All active** — filter `Active` is checked; sort Name A→Z
- **By role** — group by Role

---

## 2. Companies

| Field | Type | Options / notes |
|-------|------|-----------------|
| **Company Name** | Single line text | Primary |
| **Domain** | URL | e.g. `https://northwind.example` |
| **Industry** | Single select | `SaaS`, `Agency`, `E-commerce`, `Healthcare`, `Fintech`, `Other` |
| **Company Size** | Single select | `1-10`, `11-50`, `51-200`, `201-1000`, `1000+` |
| **Status** | Single select | `Prospect`, `Customer`, `Churned`, `Partner`, `Disqualified` |
| **Tier** | Single select | `SMB`, `Mid-Market`, `Enterprise` |
| **Owner** | Link to Team Members | Prefer single |
| **Contacts** | Link to Contacts | Allow multiple |
| **Deals** | Link to Deals | Allow multiple |
| **ARR (estimate)** | Currency | **Estimate** of annual recurring revenue; blank for prospects |
| **Open Pipeline (estimate)** | Rollup | From Deals → Amount (estimate), SUM, where Stage ∉ Won, Lost |
| **Won Deal Count** | Count | Deals linked with Stage = Won |
| **Activities** | Link to Activities | Reuse the reciprocal link created by Activities.Company; allow multiple. Do not create a duplicate reverse link. |
| **Last Activity Date** | Rollup | Companies.Activities → Date; aggregation formula `MAX(values)` |
| **Health** | Single select | `Green`, `Yellow`, `Red`, `Unknown` |
| **Tags** | Multiple select | `Inbound`, `Outbound`, `Partner-sourced`, `Expansion`, `Strategic`, `Event` |
| **Created** | Created time | System |
| **Notes** | Long text | Account context |

### Suggested formula fields — Companies

**Display ARR**
```
IF({ARR (estimate)},
  "ARR estimate: " & ROUND({ARR (estimate)}, 2),
  "ARR estimate: —"
)
```

**Needs owner?**
```
IF({Owner}, "✅", "⚠️ Unassigned")
```

### Views — Companies

- **All companies** — sort Company Name
- **Prospects** — Status = Prospect
- **Customers** — Status = Customer; group by Tier
- **Unassigned** — Owner is empty
- **Strategic accounts** — Tags contains Strategic

---

## 3. Contacts

| Field | Type | Options / notes |
|-------|------|-----------------|
| **Full Name** | Single line text | Primary |
| **Email** | Email | Unique when possible |
| **Title** | Single line text | Job title |
| **Phone** | Phone | Optional |
| **Company** | Link to Companies | Prefer single |
| **Owner** | Link to Team Members | Prefer single |
| **Role Type** | Single select | `Champion`, `Economic Buyer`, `Influencer`, `User`, `Blocker`, `Other` |
| **Lifecycle** | Single select | `Lead`, `MQL`, `SQL`, `Opportunity`, `Customer`, `Churned` |
| **LinkedIn URL** | URL | Optional |
| **Deals** | Link to Deals | Allow multiple |
| **Activities** | Link to Activities | Allow multiple |
| **Last Touched** | Date | Include time optional; keep updated via automation |
| **Do Not Contact** | Checkbox | Respect in outreach |
| **Data Quality** | Single select | `Complete`, `Needs email`, `Needs company`, `Needs enrichment` |
| **Created** | Created time | System |
| **Notes** | Long text | |

### Suggested formula fields — Contacts

**Hygiene flag**
```
IF(
  OR({Do Not Contact}, {Email} = BLANK(), {Company} = BLANK()),
  "Needs attention",
  "OK"
)
```

**Display**
```
{Full Name} & IF({Company}, " @ " & ARRAYJOIN({Company}), "")
```
*(Adjust if your Airtable plan shows linked primary values differently; ARRAYJOIN works on linked-record fields in formula.)*

### Views — Contacts

- **All contacts** — sort Full Name
- **Decision makers** — Role Type is Economic Buyer or Champion
- **Needs hygiene** — Data Quality ≠ Complete OR Hygiene flag = Needs attention
- **Do not contact** — Do Not Contact checked
- **By company** — group by Company

---

## 4. Deals

| Field | Type | Options / notes |
|-------|------|-----------------|
| **Deal Name** | Single line text | Primary; e.g. `Northwind — Pro annual` |
| **Company** | Link to Companies | Prefer single |
| **Primary Contact** | Link to Contacts | Prefer single |
| **Owner** | Link to Team Members | Prefer single |
| **Stage** | Single select | See stage list below |
| **Amount (estimate)** | Currency | **Estimate** of deal value |
| **Probability %** | Number | 0–100; integer; default by stage (see automation) |
| **Weighted Amount (estimate)** | Formula | See below |
| **Close Date** | Date | Expected or actual close |
| **Source** | Single select | `Inbound`, `Outbound`, `Referral`, `Partner`, `Expansion`, `Event`, `Other` |
| **Product Interest** | Multiple select | `Lite`, `Pro`, `Agency`, `Add-on`, `Services` |
| **Next Step** | Single line text | Short plain-language next action |
| **Next Step Due** | Date | |
| **Activities** | Link to Activities | Allow multiple |
| **Stage Changed At** | Last modified time | Watch only Stage; include time. Helper for the timestamp automation. |
| **Stage Entered** | Date | Include time; set from Stage Changed At on Stage changes (automation) |
| **Days in Stage** | Formula | See below |
| **Age (days)** | Formula | See below |
| **Forecast Category** | Single select | `Pipeline`, `Best Case`, `Commit`, `Closed` |
| **Lost Reason** | Single select | `Price`, `Competitor`, `Timing`, `No decision`, `Fit`, `Champion left`, `Other` — only when Lost |
| **Created** | Created time | System |
| **Last Modified** | Last modified time | System |
| **Notes** | Long text | |

### Stage list (single select order)

Use this order in the field config so Kanban left→right reads naturally:

1. `Qualification`
2. `Discovery`
3. `Proposal`
4. `Negotiation`
5. `Verbal Commit`
6. `Won`
7. `Lost`

Suggested default **Probability %** by stage (illustrative estimates, tune for your motion):

| Stage | Probability % (estimate) |
|-------|--------------------------|
| Qualification | 10 |
| Discovery | 25 |
| Proposal | 40 |
| Negotiation | 60 |
| Verbal Commit | 80 |
| Won | 100 |
| Lost | 0 |

### Formula fields — Deals

**Weighted Amount (estimate)**
```
IF(
  AND(LEN({Amount (estimate)} & "") > 0, LEN({Probability %} & "") > 0),
  {Amount (estimate)} * ({Probability %} / 100),
  BLANK()
)
```

**Age (days)**
```
DATETIME_DIFF(TODAY(), {Created}, 'days')
```

**Days in Stage**
```
IF(
  {Stage Entered},
  DATETIME_DIFF(TODAY(), {Stage Entered}, 'days'),
  BLANK()
)
```

**Stale?** (no next step due, or overdue, and not closed)
```
IF(
  OR({Stage} = "Won", {Stage} = "Lost"),
  "Closed",
  IF(
    OR(
      {Next Step Due} = BLANK(),
      IS_BEFORE({Next Step Due}, TODAY()),
      {Days in Stage} > 14
    ),
    "Stale",
    "On track"
  )
)
```

**Close month label**
```
IF({Close Date}, DATETIME_FORMAT({Close Date}, 'YYYY-MM'), "No close date")
```

### Views — Deals

- **Pipeline Kanban** — Group by Stage; hide Won/Lost (or show in separate views); cover Amount (estimate)
- **Closing this month** — Close Date is within this month; Stage not Lost
- **My open deals** — filter by Owner (use personal view / filter by linked user email if you map owners to Airtable collaborators)
- **Stale deals** — Stale? = Stale; Stage not Won/Lost
- **Won** — Stage = Won; sort Close Date DESC
- **Lost** — Stage = Lost; group by Lost Reason
- **Forecast — Commit** — Forecast Category = Commit
- **High value (estimate)** — Amount (estimate) ≥ 10000 (threshold is an **estimate** example; change to fit your ACV)

---

## 5. Activities

| Field | Type | Options / notes |
|-------|------|-----------------|
| **Subject** | Single line text | Primary |
| **Type** | Single select | `Call`, `Email`, `Meeting`, `Task`, `Note`, `Demo` |
| **Status** | Single select | `Planned`, `Done`, `Canceled`, `Skipped` |
| **Date** | Date | Include time recommended |
| **Owner** | Link to Team Members | Prefer single |
| **Contact** | Link to Contacts | Optional |
| **Company** | Link to Companies | Optional; or look up from Contact |
| **Deal** | Link to Deals | Optional |
| **Outcome** | Long text | What happened / summary |
| **Follow-up Needed** | Checkbox | |
| **Follow-up Due** | Date | |
| **Created** | Created time | System |

### Formula fields — Activities

**Overdue?**
```
IF(
  AND({Status} = "Planned", IS_BEFORE({Date}, TODAY())),
  "Overdue",
  IF({Status} = "Planned", "Upcoming", {Status})
)
```

### Views — Activities

- **Today** — Date is today
- **Overdue** — Overdue? = Overdue
- **This week** — Date is within this week
- **By deal** — group by Deal
- **Open follow-ups** — Follow-up Needed checked; Status ≠ Canceled

---

## Linked-record map

```
Team Members 1──* Deals
Team Members 1──* Contacts
Team Members 1──* Activities
Companies    1──* Contacts
Companies    1──* Deals
Contacts     *──* Deals          (optional many-to-many; Primary Contact on Deal is the main link)
Deals        1──* Activities
Contacts     1──* Activities
Companies    1──* Activities     (optional direct link)
```

---

## Interfaces (optional Pro+)

If you use Airtable Interfaces:

1. **Rep home** — My open deals Kanban + Today's activities + Stale deals list  
2. **Manager forecast** — Commit / Best Case / Pipeline grouped summary (Amounts are **estimates**)  
3. **Account page** — Company record with Contacts, Deals, Activities lists  

---

## Naming conventions

- Currency fields that are not settled accounting figures end with **`(estimate)`**
- Prefer short Stage names without emoji (agents and automations parse them cleanly)
- Deal Name pattern: `{Company} — {product or motion}`

