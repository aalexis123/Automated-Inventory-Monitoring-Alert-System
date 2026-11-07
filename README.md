# Automated Inventory Monitoring & Alert System

**Python · Pandas · Google Sheets · n8n · Gmail · Power BI**

## Overview
This portfolio project processes inventory records, flags products below their reorder points, and uses an n8n workflow to email inventory alerts while recording which alerts have been sent. An interactive HTML dashboard demonstrates inventory reporting; Power BI measures and a build guide are also provided.

## Business problem
Manual stock reviews can overlook shortages and lead to repetitive follow-up emails. This project combines data validation, inventory rules, scheduled notifications, and reporting in a reproducible demonstration.

## Architecture
```text
Inventory CSV → Python / Pandas notebook → Google Sheets (Inventory tab)
                                             ├→ n8n daily schedule (8 AM)
                                             │   → Read Inventory → Evaluate Stock
                                             │   ├→ New alert? → Gmail → Record alert_sent / last_alerted
                                             │   └→ Stock recovered? → Clear alert_sent
                                             └→ Dashboard data export → HTML dashboard / Power BI
```

**Important:** The Python notebook is run separately; the n8n schedule does **not** automatically execute it. The dashboard uses an exported inventory snapshot, not a live n8n connection. The exported n8n workflow is marked inactive; activate it in your own n8n instance after configuration and testing.

## Inventory rules
- **In Stock:** `quantity > reorder_point`; no new alert.
- **Low Stock:** `0 < quantity <= reorder_point`; alert if not already sent.
- **Out of Stock:** `quantity == 0`; urgent alert if not already sent.
- **Data issues:** The Python notebook validates invalid or missing fields. The n8n `Evaluate Stock` expressions independently cast quantities with `Number(...)`; improve this by explicitly filtering invalid data before sending alerts.
- **Deduplication:** On successful Gmail node execution, Google Sheets is updated with `alert_sent=TRUE` and `last_alerted`. After recovery, the reset branch clears the flag so a later shortage can alert again.

## Dashboard snapshot
The provided snapshot has **50 products**, including **11 low-stock** and **6 out-of-stock** items. These are example dataset results, not measured business impact. Open `dashboard.html` in a browser for an interactive view.

## Repository contents
```text
inventory_portfolio/
├── README.md
├── dashboard.html                    # interactive standalone dashboard
├── assets/dashboard-preview.png     # dashboard screenshot
├── data/inventory_dashboard.csv      # inventory snapshot
├── data/inventory_dashboard.json
├── python/inventory_processing.ipynb # sanitized, outputs cleared
├── n8n/Daily_Inventory_Monitor_SANITIZED.json
├── n8n/README.md
└── powerbi/
    ├── Inventory_Measures.dax
    └── POWER_BI_BUILD_GUIDE.md
```

## How to reproduce
1. Open `python/inventory_processing.ipynb` in Google Colab or Jupyter. Provide an input inventory CSV with the expected columns (see notebook). In Colab, authorize Google Sheets access and run the cells in order. The notebook computes stock status, inventory value, suggested reorder quantity, and estimated reorder cost, then writes to the `Inventory` tab.
2. Import `n8n/Daily_Inventory_Monitor_SANITIZED.json` into n8n. Reconnect **your own** Google Sheets and Gmail credentials. Replace `YOUR_SPREADSHEET_ID`, select the Inventory sheet, and replace `YOUR_ALERT_EMAIL@example.com`.
3. Run a manual test before activating the schedule. Confirm that a low-stock item triggers an email, then sets `alert_sent=TRUE` and `last_alerted`. Run again and verify no duplicate. Replenish a product and verify that its flag resets.
4. Open `dashboard.html` to view the snapshot. To create a native Power BI report, load `data/inventory_dashboard.csv`, rename the table as directed in the Power BI guide, and add the measures from `powerbi/Inventory_Measures.dax`.

## Validation and limitations
- The uploaded n8n JSON documents the workflow configuration; it does not by itself prove successful execution. Add your own **redacted** n8n execution screenshot and a sample Gmail alert to demonstrate testing.
- Google Sheets data refresh is a separate Python/Colab step, not part of the n8n schedule.
- The existing n8n numeric checks may classify blank quantities as zero. Add explicit data-quality checks before production use.
- Email sends and Google Sheets updates are separate operations; if the update fails after a successful send, a later run may send again. Production hardening should add retry/exception handling and idempotency.
- A native `.pbix` file is not included. The package provides a working HTML dashboard, data, DAX, and Power BI instructions.

## Suggested demo assets
Add `assets/n8n-workflow.png`, `assets/alert-email-redacted.png`, and `assets/execution-success.png` after capturing and redacting them. Do not publish live email addresses, spreadsheet URLs, OAuth credentials, tokens, or customer data.

## Interview summary
> I built an inventory monitoring workflow that uses Python and Pandas to validate and classify stock, Google Sheets to store the processed inventory, and n8n to schedule Gmail alerts, prevent duplicate notifications, and reset alert flags after restocking. I also created an inventory reporting dashboard and documented how to recreate it in Power BI.
