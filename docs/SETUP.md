# Setup Guide - Argentina Dollar Data Pipeline

Step-by-step instructions to get the pipeline running.

---

## 1. Install n8n

You have two options. **n8n Cloud** is the easiest to start; **self-hosted** is free forever.

### Option A - n8n Cloud (easiest)
1. Go to https://n8n.io and sign up for a free trial account
2. You get a hosted n8n instance with a URL like `https://yourname.app.n8n.cloud`
3. No installation needed - skip to step 2

### Option B - Self-hosted with npx (free, needs Node.js)
1. Install Node.js from https://nodejs.org (LTS version)
2. Open a terminal and run:
   ```bash
   npx n8n
   ```
3. n8n starts locally at `http://localhost:5678`
4. Leave that terminal open while you work

---

## 2. Import the Workflow

1. Open your n8n instance in the browser
2. Click **Add workflow** (or the **+** button)
3. Click the **three-dot menu** (top right) -> **Import from File**
4. Select `workflow/argentina_dollar_etl.json` from this repository
5. The 4-node workflow appears on the canvas

If any node shows a small warning icon after import, open it and re-select the node type or reconnect it - this can happen if your n8n version differs slightly. The logic stays the same.

---

## 3. Create the Destination Google Sheet

1. Go to https://sheets.google.com and create a new blank spreadsheet
2. Name it something like `argentina-dollar-history`
3. In the first row, add these column headers exactly:
   ```
   fecha | casa | nombre | moneda | compra | venta | spread | ingested_at
   ```
4. Copy the **Sheet ID** from the URL. The URL looks like:
   ```
   https://docs.google.com/spreadsheets/d/THIS_IS_THE_SHEET_ID/edit
   ```

---

## 4. Connect Google Sheets Credentials in n8n

1. In the workflow, open the **Append to Historical Sheet** node
2. In the **Credential** dropdown, click **Create New Credential**
3. Choose **Google Sheets OAuth2 API**
4. Follow the prompts to sign in with your Google account and authorize n8n
5. Back in the node, set:
   - **Document**: paste your Sheet ID (replace `YOUR_GOOGLE_SHEET_ID_HERE`)
   - **Sheet**: select your sheet tab (usually `Sheet1`)
   - **Operation**: Append
   - **Mapping**: Map Automatically

> n8n Cloud handles the Google OAuth setup smoothly. For self-hosted, you may need to create a Google Cloud OAuth client first - n8n shows a guide link in the credential screen if so.

---

## 5. Test the Workflow

1. Click **Test workflow** (bottom of the canvas) or **Execute Workflow**
2. Watch each node turn green as it runs
3. Open your Google Sheet - you should see one row per dollar type appended (oficial, blue, mep, ccl, etc.)
4. If the HTTP node returns data but the Code node errors, open the Code node and check the input shape matches - the transform is defensive but API responses can change

---

## 6. Activate the Schedule

1. Once the test run works, toggle the workflow to **Active** (top right switch)
2. From now on it runs automatically every day at 18:00
3. The historical dataset grows by ~7 rows per day (one per dollar type)

---

## 7. Capture Screenshots for the README

For the portfolio README, capture:

1. **Workflow canvas** - the full 4-node workflow. Save as `docs/screenshots/workflow_canvas.png`
2. **Execution result** - after a successful run, screenshot the canvas with green checkmarks, or the output panel showing the transformed rows. Save as `docs/screenshots/execution_result.png`

Then commit and push:
```bash
git add docs/screenshots/
git commit -m "docs: add workflow screenshots"
git push
```

---

## Troubleshooting

**The HTTP Request node returns nothing**
The dolarapi.com endpoint is public and stable. Check your internet connection and that the URL is exactly `https://dolarapi.com/v1/dolares`.

**The Code node errors on `$input.all()`**
Make sure the HTTP Request node runs before it and actually returned data. Run the HTTP node alone first to inspect its output shape.

**Google Sheets node fails with a permissions error**
Re-check the OAuth credential - the Google account you authorized must have edit access to the target sheet.

**Schedule does not fire**
The workflow must be toggled **Active**. Inactive workflows only run on manual test executions.
