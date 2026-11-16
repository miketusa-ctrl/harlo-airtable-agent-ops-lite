# Setup guide — Client CRM + Deal Pipeline (base a)

Brand: **Harlo**. This base **works with Airtable**. **Harlo is not affiliated with Airtable.** No Airtable logos are used in this kit.

Support: **digitalpromohub.support@gmail.com**

Time needed: about 45–90 minutes for a first-time setup, longer if you customize stages.

All sample dollars are **estimates**. Sample people and companies are fictional.

---

## Before you start

- An Airtable workspace where you can create a new base  
- Permission to create automations (and Interfaces, optional)  
- This folder: `SCHEMA.md`, `AUTOMATIONS.md`, `AGENT-RECIPES.md`, `data/*.csv`  
- Optional: an MCP-capable agent with Airtable read (and later write) tools  

---

## Step 1 — Create the base

1. In Airtable, create a new base.  
2. Suggested name: `Harlo Client CRM + Deal Pipeline`  
3. Delete any default empty table after you have created your first real table (or rename the default into **Team Members**).

---

## Step 2 — Build tables and fields

Follow `SCHEMA.md` in this order:

1. **Team Members**  
2. **Companies**  
3. **Contacts**  
4. **Deals**  
5. **Activities**  

Tips:

- Create single-select options exactly as listed (Stage order matters for Kanban).  
- Name currency fields with `(estimate)` so agents and humans stay honest.  
- Add formula fields after the columns they reference exist.  
- Linked-record fields: start from the “many” side or add both ends; Airtable will create the reverse link.

---

## Step 3 — Import seed data (optional but recommended)

1. Open `data/README.md` for import order.  
2. Import CSVs one table at a time: Team Members → Companies → Contacts → Deals → Activities.  
3. Confirm linked fields resolved (Owner, Company, Primary Contact, Deal).  
4. Spot-check: you should see open deals like `Northwind — Pro annual` and activities on overdue/planned views.  
5. When you move to real data, **delete all sample rows** first (or duplicate the base and empty it).

---

## Step 4 — Create views

Recreate the views listed in `SCHEMA.md` for each table. Minimum useful set:

| Table | Must-have views |
|-------|-----------------|
| Deals | Pipeline Kanban, Stale deals, Closing this month, Won, Lost |
| Activities | Today, Overdue, Open follow-ups |
| Contacts | Needs hygiene, Decision makers |
| Companies | Prospects, Customers, Unassigned |

---

## Step 5 — Automations

Open `AUTOMATIONS.md` and enable in the suggested turn-on order. Test each with one sample record. Turn off noisy digests until formulas and `Stale?` behave.

---

## Step 6 — Permissions

- Reps: create/edit Deals, Contacts, Activities; limited delete  
- Manager: full edit + automations  
- Avoid giving wide delete on Companies if you care about history  

If you invite collaborators that map to **Team Members**, keep the Email field identical so agents can filter “my deals.”

---

## Step 7 — Optional Interfaces

Build the three Interfaces sketched at the bottom of `SCHEMA.md` (Rep home, Manager forecast, Account page). Label any summary metrics that sum Amount fields as **estimates**.

---

## Step 8 — Connect an MCP-capable agent

1. Connect your agent’s Airtable MCP / connector to this base (read-only first).  
2. Paste the standing instructions and sample first message from `AGENT-RECIPES.md`.  
3. Run `briefing`, then `hygiene`, then `pipeline` on the sample data.  
4. Only then allow write tools for “safe fixes.”

Harlo does not ship API keys. You use your own Airtable access.

---

## Step 9 — Replace samples with production data

1. Export a backup of the sample base (Airtable snapshot / duplicate).  
2. Delete sample records.  
3. Import your Companies and Contacts; keep field names stable.  
4. Re-map Owners to real Team Members.  
5. Re-test automations 1–2 on a dummy deal.  
6. Confirm Do Not Contact and privacy practices before any outreach process uses this base.

---

## Step 10 — Smoke checklist

- [ ] Five tables present with primary fields named as in schema  
- [ ] Deal stages in the documented order  
- [ ] Weighted Amount (estimate) formula returns values on open deals  
- [ ] Stale? formula populates  
- [ ] Kanban view groups by Stage  
- [ ] At least Automations 1–4 tested  
- [ ] Agent briefing returns real record names (not invented)  
- [ ] Disclaimer understood: not affiliated with Airtable; dollars are estimates  

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Linked CSV fields blank | Import parent table first; primary text must match exactly |
| Weighted Amount blank | Amount (estimate) or Probability % empty |
| Stale? always Stale | Set Next Step Due; confirm Stage Entered automation |
| Agent invents deals | Tighten standing instructions; require tool results only |
| Automation loops | Don’t watch fields the same automation writes without conditions |

---

## What this kit is not

- Not a hosted CRM  
- Not legal or revenue recognition advice  
- Not an official Airtable product or partner pack  
- Not permission to email the fictional `@*.example` seed addresses  

Questions about the kit: **digitalpromohub.support@gmail.com**

## Table shells and reciprocal links

Create all named table shells first, then configure fields. When a linked-record field creates a reciprocal field, rename and reuse that field rather than creating a second link. Import parent records before dependent records. Verify linked values resolve to separate existing records after import.

In Activities, configure Company → Companies. Rename its automatically created reverse field in Companies to Activities. Configure Companies.Last Activity Date as a rollup of Activities → Date, using `MAX(values)`. It is not a multi-hop rollup through Contacts or Deals.
