# Automations — Client CRM + Deal Pipeline (base a)

Step-by-step recipes for Airtable’s built-in Automations. Harlo is **not affiliated with Airtable**. These steps describe how the base **works with Airtable** automation features.

Dollar amounts referenced in messages are **estimates**.

Test each automation with a throwaway record before enabling it on the full base.

---

## Automation 1 — Stamp Stage Entered when Stage changes

**Goal:** Keep `Days in Stage` accurate.

1. Add Deals helper **Stage Changed At**: type **Last modified time**, watch only **Stage**, include time. Configure Stage Entered as a date with time enabled.
2. Create automation `Deal — stamp Stage Entered`.
3. Trigger: **When a record is updated**, table **Deals**, watched fields: **Stage only**. Do not use “matches conditions / Stage not empty”: that will not fire on consecutive nonempty stages.
4. Action: **Update record**, table Deals, Record ID = trigger Airtable record ID. Set **Stage Entered** using the trigger record's **Stage Changed At** field token (select the field from dynamic values, not literal text). This is a real last-modified timestamp; do not type `NOW()` into a date action.
5. Test Proposal → Negotiation, then Negotiation → Verbal Commit at distinguishable times. Inspect both runs: Stage Entered must equal Stage Changed At each time; Days in Stage must reset. Only Stage is watched, so updating Stage Entered does not loop.
6. Initialization is separate: imported seeds already have Stage Entered dates. For new records, either enter the initial Stage Entered date/time manually or create a separate **When record created** automation using the trigger's **Created** timestamp token. The field-change automation handles all later transitions.

These steps are implementation instructions; no live Airtable execution is claimed by this file.

---

## Automation 2 — Default Probability % from Stage

**Goal:** Keep weighted pipeline estimates consistent.

1. Name: `Deal — default probability by stage`
2. **Trigger:** When a record is updated → Deals → watch `Stage`
3. **Action:** Conditional logic (one branch per stage) **or** separate automations per stage if conditionals are limited:

| If Stage equals | Set Probability % (estimate defaults) |
|-----------------|----------------------------------------|
| Qualification | 10 |
| Discovery | 25 |
| Proposal | 40 |
| Negotiation | 60 |
| Verbal Commit | 80 |
| Won | 100 |
| Lost | 0 |

4. On **Won** branch also set `Forecast Category` → `Closed`  
5. On **Lost** branch set `Forecast Category` → `Closed` and leave `Lost Reason` for the rep to fill  
6. Test: move a deal Qualification → Discovery; Probability % should become 25.  
7. **Note:** Reps may override Probability %. If you want overrides preserved, add a checkbox `Lock probability` and skip the update when checked.

---

## Automation 3 — Won deal → Company becomes Customer

**Goal:** Account status stays in sync when revenue closes.

1. Name: `Deal Won — mark Company Customer`
2. **Trigger:** When record matches conditions → Deals  
   - `Stage` is `Won`
3. Or: When updated → watch `Stage` → condition Stage = Won
4. **Action:** Update record on **Companies**  
   - Record ID: from trigger → linked `Company`  
   - Set `Status` → `Customer`  
   - Optionally set `Health` → `Green` if it was `Unknown`
5. **Action (optional):** Find records → Contacts where Company = that company AND Lifecycle ≠ Customer → Update Lifecycle → `Customer` (batch carefully)
6. Test on a fictional deal only.

---

## Automation 4 — New deal → create kickoff Task activity

**Goal:** Every new opportunity gets a next step.

1. Name: `Deal created — kickoff task`
2. **Trigger:** When record created → Deals
3. **Action:** Create record → **Activities**  
   - Subject: `"Kickoff: "` & Deal Name *(use field tokens)*  
   - Type: `Task`  
   - Status: `Planned`  
   - Date: 2 days from now (or `TODAY()` + use a script/formula helper if date math is limited — otherwise set Date to trigger time and put due language in Subject)  
   - Owner: same as Deal → Owner  
   - Deal: link trigger deal  
   - Company: link trigger Company  
   - Contact: link Primary Contact  
   - Follow-up Needed: checked  
4. **Action:** Update Deal → `Next Step` = `Complete kickoff task` ; `Next Step Due` = same date  
5. Test: create a deal; confirm Activity appears.

---

## Automation 5 — Proposal stage → schedule follow-up

**Goal:** Proposals do not go dark.

1. Name: `Deal → Proposal follow-up`
2. **Trigger:** When updated → Deals → watch Stage → condition Stage = `Proposal`
3. **Action:** Create Activity  
   - Subject: `Proposal follow-up — {Deal Name}`  
   - Type: `Email`  
   - Status: `Planned`  
   - Date: 3 business days out (pick a fixed +3 calendar days if business-day logic unavailable)  
   - Owner / Deal / Contact / Company from trigger  
   - Follow-up Needed: checked  
4. **Action:** Update Deal → Next Step = `Send or confirm proposal follow-up` ; Next Step Due = that date  
5. Test by moving a seed deal into Proposal.

---

## Automation 6 — Stale open deal Slack/email nudge (optional)

**Goal:** Surface deals with `Stale?` = Stale. *(Requires a notification destination you already use.)*

1. Name: `Daily — stale deal digest`
2. **Trigger:** At a scheduled time → Daily → e.g. 8:00 AM in your workspace timezone  
3. **Action:** Find records → Deals  
   - Conditions: `Stale?` is `Stale` AND Stage is none of `Won`, `Lost`  
4. **Action:** Send email *(or Slack if connected)*  
   - To: Manager email or a shared inbox you control  
   - Subject: `Harlo CRM — stale deals digest (estimates)`  
   - Body: list Deal Name, Owner, Amount (estimate), Days in Stage, Next Step  
5. If Find returns 0 records, either skip send (if “only if records found” is available) or send “No stale deals today.”  
6. Keep recipients internal. Do not email sample `.example` addresses from the seed CSVs.

---

## Automation 7 — Activity Done → bump Contact Last Touched

**Goal:** Hygiene for “who did we actually talk to?”

1. Name: `Activity Done — touch Contact`
2. **Trigger:** When updated → Activities → watch `Status` → condition Status = `Done`
3. **Condition:** Contact is not empty
4. **Action:** Update Contacts → Last Touched = Date from Activity (or NOW())  
5. Optional second action: Update linked Company Notes is usually too noisy — skip unless you want a Last Activity Date field maintained the same way.

---

## Automation 8 — Contact missing email or company → flag Data Quality

**Goal:** Feed the hygiene agent and the Needs hygiene view.

1. Name: `Contact — data quality flag`
2. **Trigger:** When record created or updated → Contacts → watch Email, Company, Do Not Contact  
3. **Conditional actions:**  
   - If Do Not Contact checked → set Data Quality = `Complete` *(or leave as-is; still exclude from outreach views)*  
   - Else if Email empty → `Needs email`  
   - Else if Company empty → `Needs company`  
   - Else → `Complete`  
4. Test with a contact that has no email.

---

## Automation 9 — Lost deal requires Lost Reason (soft gate)

Airtable cannot hard-block a field easily without Interface rules. Soft approach:

1. Name: `Deal Lost — remind Lost Reason`
2. **Trigger:** Stage becomes `Lost`
3. **Action:** If Lost Reason is empty → Send email / in-Airtable notification to Owner:  
   `Please set Lost Reason on {Deal Name}`  
4. Optional: Create Activity Type Note Subject `Lost — reason TBD`

---

## Suggested turn-on order

1. Stage Entered stamp  
2. Probability defaults  
3. Kickoff task on create  
4. Activity → Last Touched  
5. Contact data quality  
6. Won → Customer  
7. Proposal follow-up  
8. Stale digest (after formulas/views exist)  
9. Lost reason nudge  

---

## Safety notes

- Run automations as a dedicated workspace user when possible so history is auditable.  
- Disable seed-data tests before importing production data.  
- Never auto-email real customers from these templates without a human approval step.  
- Amounts in notification text are **estimates**.
