---
name: "weekly-sales-report"
description: "Every week, build the weekly sales report from the 판매관리 Google Sheet for the previous Monday–Sunday and post it to Slack (one-line header + details in thread). Use for the weekly sales report or its recurring run."
---

# Weekly Sales Report (위클리 세일즈 리포트)

This report is produced **every week**. Each run covers the most recent full week (Monday–Sunday) and compares it with the week before. It reads the Google Sheet `판매관리` and posts to Slack: a one-line header in the channel, details in that message's thread.

The report is meant to be repeatable: the settings in section 0 were confirmed by the user and are reused every week without asking again. Ask only when the user wants to change them, or when something does not fit (see section 2).

## 0. Confirmed settings (defaults for every run)

- **Channel**: #<YOUR_CHANNEL> (channel_id <YOUR_CHANNEL_ID>). Change here if the user moves the report to another channel.
- **Source file**: Google Sheet `판매관리`, sheet `판매내역`. Two files share this name; use the one that contains rows for the target week. Current file: owner <SHEET_OWNER_EMAIL> (id <YOUR_SHEET_ID>).
- **Basis**: payment amount (결제금액) sum, shipping fee (배송비) excluded, compared with the previous week.
- **Items**: summary (total sales, orders, quantity, average order value), sales by channel, sales by category, product TOP 5.
- **Header format**: `[주간매출리포트] 10/19(월)~10/25(일)` using the real dates, on one line.

## 1. Work out the week

Get today's date and weekday with the current-time tool. Target week = the Monday–Sunday that ended most recently (if today is Monday, it is the week that ended yesterday). Comparison week = the Monday–Sunday before that. Make sure the year is right: dates in the sheet include the year.

## 2. Read the data and check coverage

Columns of `판매내역`: 주문일, 주문번호, 판매채널, 카테고리, 상품명, 옵션, 수량, 판매단가, 결제금액, 배송비. Numbers are strings with thousands separators (e.g. `33,300`).

- Rows are sorted by date, about 20–25 rows per day, with blank rows at the end of the grid. Find the range by probing column A with small ranges (e.g. `A7400:A7402`), then fetch the comparison week + target week in one `A:J` call. Rows are added as the weeks go by, so the position moves every week; do not reuse an old range.
- Search Drive again each run (`title contains '판매관리'`, spreadsheet mime type). If more than one file matches, use the one with data for every day of the target week. If several have it, or none does, stop.
- **Coverage check**: every day of the target week and the comparison week must have rows. If data is missing, do not post a report with zeros or partial numbers. Tell the user what is missing. A file can end before the target week or contain stray rows (the user's own copy had one stray row dated after its last real day).
- If a day has no sales at all, mention it to the user rather than silently reporting it.

## 3. Compute with code, never by hand

Put the fetched rows into a Python script and compute total sales, order count (distinct 주문번호; one order can span several rows), quantity, average order value, per-channel and per-category sales with share % and change vs. the previous week, and the product TOP 5 by sales with quantity. Strip commas before summing. Re-derive each percentage once from the printed totals before writing it into the message (a rounding slip happened before: -38.05% was written as -38.1%).

## 4. Check for duplicates, then post to Slack

1. Read the channel with `slack_read_channel`. **If a message with the same header text already exists, send nothing** (a rerun must never post the same week twice). Tell the user.
2. Send the header line with `slack_send_message`.
3. Send the details with `slack_send_message` using `thread_ts` of the header message (from the returned message link / `message_ts`). Order: summary (total sales with change vs. previous week, orders, quantity, average order value), sales by channel table (with previous-week sales and change), sales by category table (same), product TOP 5, and a source line naming the sheet and comparison week.
4. Slack messages cannot be edited with the available tools, so verify numbers before step 2.

## 5. Running every week

- **Schedule**: the report is meant to go out weekly, e.g. every Monday morning (Asia/Seoul) after the week has closed. If the user has not set up a scheduled task yet, offer to create one with the scheduling tools. The prompt of the scheduled task must be self-contained: "Run the weekly-sales-report skill for last week (Mon–Sun) and post it to the channel in the skill's settings."
- **Unattended runs**: nobody is there to answer questions. Use the settings in section 0 and do not ask. If a condition in section 2 is not met (missing data, ambiguous file), post nothing and report the problem in the final message instead. The duplicate check in section 4 makes retries safe.
- **Changing the report**: when the user asks for different items, basis, or channel, apply the change for that run and update section 0 of this skill so later weeks use it.

## 6. Report back

Say in a few lines what was posted and where, the headline numbers, which file and basis were used, and any caveat (duplicate file names, missing data, rounding corrections).