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

<img width="1750" height="772" alt="image" src="https://github.com/user-attachments/assets/b20e7684-7903-4f0e-b21a-db8b0d057892" />

## n8n Workflow Automation
I developed a daily inventory monitoring workflow using n8n, Google Sheets, and Gmail to automate stock evaluation and notification tracking.

The workflow is configured to check inventory each morning at 8:00 AM and identify products that have reached or fallen below their reorder thresholds.

Workflow Screenshot
<img width="1858" height="601" alt="image" src="https://github.com/user-attachments/assets/918c5496-4ec8-427e-9851-af37a0a882f2" />


Workflow Architecture
```text
Daily Check (8 AM)
        |
        v
Read Inventory (Google Sheets)
        |
        v
Evaluate Stock
        |
        +------------------------+
        |                        |
        v                        v
Needs New Alert           Stock Recovered
        |                        |
        v                        v
Send Stock Alert          Clear Alert Flag
        |
        v
Record Alert Sent
```
Step 1: Daily Inventory Check

Nodes: Daily Check → Read Inventory

The workflow begins with a scheduled trigger configured for 8:00 AM.

It retrieves inventory records from Google Sheets, including product quantities, reorder thresholds, and existing alert statuses.

Step 2: Inventory Evaluation

Node: Evaluate Stock

JavaScript expressions evaluate each product and determine whether it is:

In Stock

Low Stock

Out of Stock

The workflow also determines whether a new notification is required or an existing alert flag should be reset.

Step 3: Automated Stock Notifications

Nodes: Needs New Alert → Send Stock Alert

When a product reaches or falls below its reorder threshold, the workflow checks whether an alert has already been sent.

If a new alert is required, Gmail sends an automated notification containing:

Product name

Current inventory quantity

Reorder threshold

Inventory status

Out-of-stock products receive an urgent restock notification.

Automated Email Screenshot



Step 4: Duplicate Alert Prevention

Node: Record Alert Sent

After sending a notification, the workflow updates Google Sheets:

alert_sent = TRUE
last_alerted = Current Timestamp

This prevents the workflow from repeatedly notifying users about the same unresolved inventory shortage.

Alert Tracking Screenshot



Step 5: Automatic Alert Reset

Nodes: Stock Recovered → Clear Alert Flag

When a previously flagged product is restocked above its reorder threshold, the workflow clears its alert status:

alert_sent = FALSE
last_alerted = Blank

This allows a future shortage to trigger another notification.

Successful Workflow Execution

## Project Outcomes

This project demonstrates a practical approach to combining data engineering, business intelligence, and workflow automation.

Key capabilities implemented include:

Processing and classifying inventory records using Python.

Identifying low-stock and out-of-stock products.

Calculating inventory values and estimated replenishment costs.

Building an automated inventory notification workflow.

Preventing repeated notifications for unresolved shortages.

Resetting alerts when inventory recovers.

Creating a dashboard for monitoring inventory performance.

The system provides a foundation for reducing manual inventory checks and improving the visibility of replenishment needs.
