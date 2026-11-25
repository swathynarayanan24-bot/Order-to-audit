# Order-to-audit
This is a project specifically implemented to secure a internship position in a prestigious institution, NIT Tiruchirappalli where I designed an order to audit system
Murukku Works · Order-to-Audit
A working prototype that takes a small South Indian snack factory from a salesman's order to the auditor's monthly report. Every transaction is entered once and flows into stock, production, costing and accounts.

Two loops meeting at finished goods:

Sales: order → approval → GST invoice → dispatch → payment

Production: purchase → issue per BOM → run → finished goods → variance and cost per packet

📖 Table of contents
Quick start

Stack and why

Features against the tiers

Benchmark numbers

Design note

Assumptions

Known gaps

🚀 Quick start
Pick how you want to run it. No install or build step.

<details open> <summary><b>🖱️ Open directly</b></summary>
Double-click order-to-audit-colorful.html. Data is kept in your browser's localStorage.

</details><details> <summary><b>💻 Local server</b></summary>
bash
python3 -m http.server 8000
# then open http://localhost:8000/order-to-audit-colorful.html
</details><details> <summary><b>☁️ Host it</b></summary>
bash
git init && git add . && git commit -m "prototype"
git remote add origin <your-repo-url> && git push -u origin main
# enable GitHub Pages on the main branch
</details>
🔑 Test logins
Use the "Signed in as" switcher in the sidebar. There are no passwords in this prototype.

Role	Sees	Copy ID
Salesman A / B	New order, My orders (own only)	salA
Factory Admin	Orders, Invoices & pay, Stock, Reports	admin
Production Supervisor	Purchase, Production, Stock	prod
Accountant	Reports, Stock (read-only)	acct
Owner	Dashboard, Stock, Reports	owner
⚡ Fastest check: press ▶ Play benchmark in the sidebar and watch section 5 run end to end.

🧱 Stack and why
One HTML file with vanilla JavaScript and browser storage. It needs no server, so an evaluator can run it in seconds, and it works on a weak mobile network once loaded. Money is integer paise and quantities are integers in grams, millilitres or pieces, so nothing drifts through rounding.

⚠️ Honest trade-off: this does not meet the "real database" expectation. The ledger design (below) maps directly to Postgres or SQLite. See Known gaps.

✅ Features against the tiers
Progress at a glance

Tier	Progress
Must	████████████████████ 100%
Should	██████████████████░░ ~92%
Stretch	██░░░░░░░░░░░░░░░░░░ ~9%
Tech	████████████░░░░░░░░ ~63%
<details open> <summary><b>Must</b></summary>
Feature	Status	Note
Login with roles	🟡 Partial	User switcher, no passwords
Product, customer, raw material, BOM masters	🟡 Partial	Seeded, no create screens
Mobile order entry + admin approval	🟢 Done	
GST invoice as PDF	🟢 Done	Print to PDF
Production run updates stock	🟢 Done	
Planned vs actual + variance	🟢 Done	
Stock summary + ledger	🟢 Done	
Sales register + stock report CSV	🟢 Done	
</details><details> <summary><b>Should</b></summary>
Feature	Status	Note
Purchases and suppliers	🟡 Partial	Supplier is free text
Payments, outstanding, ageing	🟡 Partial	No ageing buckets
Cost per packet with overheads	🟢 Done	
GST summary report	🟢 Done	GSTR-1 style
Variance alerts	🟢 Done	
Owner dashboard	🟢 Done	
</details><details> <summary><b>Stretch</b></summary>
Feature	Status	Note
Offline order capture	🔴 Not built	
Batch and expiry, FEFO	🔴 Not built	
Reorder / low-stock alerts	🟡 Partial	Dashboard shows raw stock under 5 units
Production planning from orders	🔴 Not built	
GPS visit log	🔴 Not built	
Tally export	🔴 Not built	
e-Invoice / e-Way Bill	🔴 Not built	
Tamil interface	🔴 Not built	
</details><details> <summary><b>Tech</b></summary>
Feature	Status	Note
Real database	🔴 Not built	Browser storage only
Ledger-based stock, integer money	🟢 Done	
Audit trail	🟡 Partial	Actions logged; no credit note
Seed data + one-click benchmark	🟢 Done	Play benchmark
</details>
📊 Benchmark numbers
Change the inputs in the interactive prototype and see every checked figure recompute. Defaults are the section 5 scenario. Costing method is switchable.

Check	Expected	Prototype	Result
Flour closing	120 kg	120 kg	✅
Oil closing	19.5 L	20 L (default)	✅
Pouches closing	306	306	✅
FG after run	238 pkts	238 pkts	✅
FG after Retailer A	88 pkts	88 pkts	✅
Yield variance	−6%, flagged	−6.0%, flagged	✅
Oil variance	+5%, flagged	+5.0%, flagged	✅
Retailer A invoice	₹7,875.00	₹7,875.00	✅
Retailer A outstanding	₹3,875.00	₹3,875.00	✅
Distributor B shortage	32 packets	32 packets	✅
🤔 Why numbers differ or go beyond the sheet
<details> <summary><b>Cartons show −10% and are flagged</b></summary>
9 used against a standard of 10. The system flags under-use as well as over-use.

</details><details> <summary><b>Pouches sit exactly at −3% and are not flagged</b></summary>
194 used against 200. The rule flags only variance beyond the 3% tolerance.

</details><details> <summary><b>Cost per packet is our own figure</b></summary>
It uses weighted-average valuation. Labour and power are charged straight to the run and divided by good packets. Try the other methods (last price, FIFO) in the prototype to see the sensitivity.

</details>
🧮 Design note
How stock is computed
text
closing = Σ ledger.q  (opening + receipts − issues ± adjustments)
issue value = item.value × qty_issued / balance   // weighted average
production: issue each BOM line, then post FG receipt valued at materials cost
Nothing overwrites a balance. Every movement stores its source (OPENING, PURCHASE, PRODUCTION, INVOICE) and reference, and an issue larger than the balance is rejected with nothing posted.

Data model
<details> <summary><b>Click to expand the full entity list</b></summary>
Entity	Fields	Notes
Item	id, name, type (Raw/Packaging/FG), unit, running value	Referenced by Ledger, BOM, Order lines
Ledger	n, ts, item→Item, q (signed base units), v (paise), src, ref, by	Single source of truth for every balance
BOM	product→Item, yield, lines[item→Item, standard qty]	Used by Production run
Run	id, product, expected, good, damaged, lines[item, planned, actual], materials, labour, power	Posts Ledger issues and an FG receipt
Purchase	id, supplier, item→Item, qty, rate	Posts a Ledger receipt
Customer	id, name, state, state code, GSTIN	Decides CGST+SGST vs IGST
Order	id, customer, salesman, lines, status	Becomes an Invoice
Invoice	no (FY-numbered), order→Order, lines (HSN, taxable, CGST/SGST/IGST), total, dispatched, status	Dispatch posts Ledger issues
Payment	invoice→Invoice, customer, amount, time	Reduces outstanding
Audit	time, user, action	Appended on every key action
</details>
Costing rule
Materials at weighted-average cost. Labour and power are allocated per run to that run's good packets. Margin is the selling price minus cost per packet.

📐 Assumptions
GST 5%, HSN 1905. Tamil Nadu customers get CGST + SGST, other states get IGST.

Invoice numbers follow the financial year, such as INV/2026-27/0001.

Opening finished goods are valued at ₹30 per packet.

A variance beyond 3% in either direction is flagged.

Pouch consumption equals good packets plus damaged pouches.

Finished-goods ledger value carries materials cost only. Overheads appear in the run cost.

🕳️ Known gaps
<details> <summary><b>No real database or multi-user sync</b></summary>
Data is per-browser. Next step: a REST API on Postgres using the same ledger tables.

</details><details> <summary><b>No passwords or real authentication</b></summary>
Roles come from a user switcher.

</details><details> <summary><b>Invoices cannot be cancelled with a credit note yet</b></summary>
Posted invoices are never deleted, but the credit-note flow is not built.

</details><details> <summary><b>No automated tests, offline order queue, batch or expiry, or ageing buckets</b></summary>
Tests for stock and tax maths are the first priority.

</details><details> <summary><b>Master data is seeded, with no screens to create it</b></summary>
Only Murukku 200 g has a BOM.

</details>
<p align="center"> <i>Built as a working prototype — open the HTML file and press <b>▶ Play benchmark</b> to see it run.</i> </p>
