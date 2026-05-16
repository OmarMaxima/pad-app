# PAD Power Automate Flows

Six flows that handle all server-side routing, notifications, and state management for the PAD system.

## Import Instructions

1. Open Power Automate → **My flows** → **Import** → **Import Package (Legacy)**
2. Upload the `.zip` for the flow you want (or use the JSON definitions below with **Create from blank → Logic Apps code view**)
3. When prompted, map the SharePoint and Teams connections to your existing connections
4. Turn the flow on

> **Prerequisite:** You must have the SharePoint site `maximaapparel.sharepoint.com/sites/OMARTEST` and the three lists (`PAD Capsules`, `Rework History`, `Handoff Tracker`) already set up. The `Merch Tasks` list is created during Merch workflow setup.

---

## Flow Index

| File | Trigger | Purpose |
|---|---|---|
| `01-status-router.json` | PAD Capsules item modified | Routes style to next stage based on status change |
| `02-rework-logger.json` | PAD Capsules item modified (rejected) | Logs rejection to Rework History, notifies designer |
| `03-merch-task-notifier.json` | Merch Tasks item created | Sends Teams adaptive card to assignee |
| `04-deadline-alert.json` | Daily at 8:00 AM | Alerts manager + Sr. on overdue / near-due styles |
| `05-licensing-gate.json` | PAD Capsules SR_Review_Status → Approved | Advances to Licensing, notifies Licensing team |
| `06-style-number-unblock.json` | Merch Tasks item modified (new style # set) | Updates PAD Capsules, notifies designer and Merch |

---

## SharePoint Connection Details

```
Site URL:  https://maximaapparel.sharepoint.com/sites/OMARTEST
Lists:
  PAD Capsules   — main style pipeline
  Rework History — rejection log
  Handoff Tracker — approved styles
  Merch Tasks    — merch team task queue
```

## Teams Channels

Flows post to these channels. Update the channel IDs after import.

| Channel | Used by |
|---|---|
| Pro Standard > PAD Notifications | Status changes, rework alerts |
| Pro Standard > Merch | Merch task notifications |
| Pro Standard > Licensing | Licensing gate notifications |
