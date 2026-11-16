# Seed data (fictional)

All rows are **made up**. Domains use `.example`. Phone numbers are fake `555-` lines. Dollar amounts are **estimates** for demo math only.

## Import order

1. `team_members.csv`
2. `companies.csv` (Owner matches Team Members → Name)
3. `contacts.csv` (Company and Owner by name)
4. `deals.csv` (Company, Primary Contact, Owner by name)
5. `activities.csv` (Owner, Contact, Company, Deal by name)

Airtable CSV import matches linked records by primary-field text when the linked table already has those records. If a link fails, create the parent row first or paste the link after import.

Delete these sample rows before you load real customer data.

Multi-value link and select cells use CSV-quoted comma-separated values (for example, `"Casey Nguyen, Ben Holloway"`). Keep CSV quoting intact on import; each comma-separated value is a separate linked record or select option. Verify Team and Tags tokens individually after import; do not create compound-name Team records.
