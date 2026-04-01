# Tines Bootcamp: Building Your First Automation Story

Welcome to Tines! This bootcamp will introduce you to the core concepts of Tines by building a practical Alert Ingest Story.

## What You Will Build

In this training, you will build a Story to:

1. Get a list of new alerts (simulated).
2. Deduplicate incoming events to prevent processing the same alert twice.
3. Explode the array of alerts so each one is processed individually.
4. Enrich the IP address associated with the alert.
5. Summarize the alert using Automatic Mode (AI-powered).
6. Filter alerts based on severity using a Trigger action.
7. Create a Jira ticket or send an email with the formatted data.

---

## Fundamental Tines Concepts for Builders

Before starting, understanding how events flow and how to test your logic will greatly simplify the building process.

### 1. Event Flow Direction and Links

- **Default Flow (Solid Line):** Events always try to flow through the solid line link, which represents the standard success path. One action completes and passes its event to the next.
- **Logic Fork (Dotted Line):** Only the Trigger and Filter actions can create a dotted line link, which represents the "no match" path. This is crucial for splitting events into separate logical branches.
- **Failure:** If an action fails (for example, a bad API response), the failure is logged and the event stops, preventing bad data from moving downstream.

### 2. Action Run vs. Action Scheduling

- **Action Run (Manual):** When you click Run on an action, you are executing it immediately to test the logic. This is the method used throughout the bootcamp.
- **Scheduled/Webhook Run (Production):** In a live environment, a story is typically kicked off automatically via a Schedule (set on the first action) or a Webhook (receiving an external event). Once set up, manual runs are no longer required.

### 3. Testing and Event Re-emission

- **Events vs. Runs:** An event is the piece of data produced. A run is the execution of the action. These are not the same thing.
- **Re-emission:** When testing, you often click Run on a downstream action and select an upstream event to re-emit (re-process). This lets you test changes to a formula or logic block without re-running the slow external API calls that precede it.

---

## The Tines Approach to AI

Tines offers powerful AI features that run securely within your tenant's infrastructure. This bootcamp uses the **Automatic mode** feature, which leverages AI to generate complex data manipulation code from a simple prompt.

> **Gotcha: AI Feature Requirement**
> This bootcamp requires AI-powered features. Confirm that your administrator has enabled these features in your tenant settings before proceeding.

---

## Step 1: Set Up Your Credential

**Type:** Information Action

Before your Story can connect to any tools, you must create a Credential.

### Create the `bootcamp_api` Text Credential

1. Click anywhere on the storyboard to bring up the Story Menu.
2. Find the Credentials section and click the **+** button.
3. Select the **Text** credential type.
4. Set **Name** to: `bootcamp_api`
5. Set **Value** to: `secret_api_key`
6. Scroll down to **Domains** and enter: `toolkit.tines.com`
7. Click **Save**.

---

## Step 2: Get New Alerts

**Type:** Template Action

We use a preconfigured Template to simulate receiving a list of alerts.

### Add the "Get New Alerts" Action

1. Go to the **Template Library** and search for **Tines Bootcamp**.
2. Click and drag the Tines Bootcamp template onto your Storyboard.
3. Select the **Get New Alerts** action from the menu on the left.

### Run and Inspect the Action

1. With the action selected, click **Run**.
2. After the run completes, click **Events**.

> **Gotcha: Accessing Event Data**
> Single-click an event entry to select it. To expand and inspect nested data, double-click the `{...}` next to an object like `body`.

---

## Step 3: Explode the Array

**Type:** Event Transform Action

We use Explode mode to break the array of alerts into individual events so each one can be processed separately.

### Configure the "Explode Alerts" Action

1. Drag an **Event Transform** action onto the Storyboard, connect it, and rename it: `Explode Alerts`
2. Set **Mode** to **Explode**.
3. Set the **Path** to: `get_new_alerts.body.alerts`

> **Gotcha: Avoiding AI Auto-Completion**
> Do NOT select the AI Auto-completion option when setting the path. Build the path manually. The AI will automatically append a dot ( `.` ) to the end of the path, which will break it.

### Test the Explosion and Find the Duplicate

1. Click **Run** beneath the Explode Alerts action. Select the run from the "Get New Alerts" action.
2. Click **Events**. A new tab will open with the event data structured like this:

```
"get_new_alerts": {4},
"explode_alerts": {4}
```

> **Gotcha: Expanding Event Data**
> The number inside the braces represents the total count of items or properties within that data structure. Click the number to expand, or use the expand button in the upper right to see everything at once.

3. Follow this path in the event data:

```
get_new_alerts
└─ body
   └─ alerts
```

4. Note that there is more than one entry for Thomas Kinsella. After exploding, each alert becomes its own event and will traverse the story independently.

---

## Step 4: Deduplicate Events

**Type:** Event Transform Action

We use Deduplicate mode to prevent the duplicate alert from being processed more than once.

### Configure the "Deduplicate" Action

1. Drag an **Event Transform** action onto the Storyboard, connect it, and rename it: `Deduplicate`
2. Set **Mode** to **Deduplicate**.
3. Set the **Path** to: `explode_alerts.individual_item`

### Run and Inspect the Deduplication

1. Click **Run** beneath the Deduplicate action. Select the event run from the "Explode Alerts" action.
2. Click **Events**. The Deduplicate action should emit only three events.

---

## Step 5: Enrich the IP Address

**Type:** HTTP Request Action (via Curl2Tines)

We use a cURL command to fetch enrichment data for the alert's IP address.

### Create the HTTP Request Action

1. Copy this cURL command:

```bash
curl -X GET \
  'https://toolkit.tines.com/api/public/searchforIPinAbuseIPDB' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <<CREDENTIAL.bootcamp_api>>' \
  -G \
  --data-urlencode 'ipAddress=<<explode_alerts.individual_item.ip_address>>'
```

2. Paste the command directly onto the Storyboard. Tines will automatically convert it into an HTTP Request action.
3. Connect the output of **Deduplicate** to the new action.
4. Rename the action to: `Search for IP Address in Abuse IPDB`

### Run and Inspect the Enrichment

1. Click **Run** beneath the action. Select any one of the unique event runs from the "Deduplicate" action.
2. Click **Events** and confirm the `body` field contains enrichment results such as `abuseConfidenceScore`.

---

### Understanding the Data Editor Tabs

When configuring an action's Payload (or Input in Automatic mode), Tines provides three tabs for interacting with data:

| Tab | Purpose | Best Used For |
|---|---|---|
| **Builder** | The default graphical interface for constructing JSON objects, arrays, and basic formulas using key/value pairs and drag-and-drop pills. | Initial setup: creating basic data structures, inserting pills for data paths, and using simple formulas. |
| **Plain code** | Shows the raw JSON representing the data you built. The definitive source for troubleshooting complex syntax where quotes and commas may conflict. | Verification: troubleshooting complex syntax errors and manually inserting clean JSON or advanced code structures. |
| **Output** | A live preview of the action's resulting event after a run. Dynamically displays the final data that will be emitted, showing the results of your formulas or code. | Debugging: checking if a formula is calculating correctly, ensuring path manipulation worked, or seeing the exact error message. |

---

## Step 6: Summarize with Automatic Mode

**Type:** Event Transform Action

We use Automatic mode to generate the summary, title, and formatted HTML without manual coding.

### Configure the "Build Summary & HTML" Action

1. Drag an **Event Transform** action onto the storyboard and connect it to **Search for IP Address in Abuse IPDB**.
2. Rename it: `Build Summary & HTML`
3. Set the **Mode** dropdown to **Automatic**.
4. Click the **Input** field to open the configuration modal.

### Provide the Prompt

1. In the Input configuration modal, set the upstream data path to:
   - **Path:** `search_for_ip_address_in_abuse_ipdb`

2. In the Prompt area, paste the following:

```
Generate a JSON event with four keys:

1. alert_title: Concatenate the text 'Priority Alert for: ' with the user's email.
2. risk_rating: Assign a risk of 'HIGH (9/10)' if the IP's confidence score is above 50, 'MEDIUM (6/10)' if the score is above 10, and 'LOW (3/10)' otherwise.
3. alert_summary: Write a 2-3 sentence summary detailing the suspicious login, including the system, IP address, and the calculated risk rating.
4. html_body: Generate a simple HTML table containing the User Email, System, IP Address, and the calculated Risk Rating.

The primary alert data is from the upstream action explode_alerts.individual_item and the enrichment data is from search_for_ip_address_in_abuse_ipdb.body.data.
```

3. Click **Generate**, then click **Save** to commit the Python code.

---

## Step 7: Configure a Trigger

**Type:** Flow Control Action

This action splits the workflow based on the original alert severity.

### Add and Configure the Trigger

1. Add the **Trigger** action and connect it to **Build Summary & HTML**.
2. Rename it: `Alert Severity is High`
3. Set the rule configuration:
   - **Path:** `explode_alerts.individual_item.severity`
   - **Condition:** is equal to
   - **Value:** `high`

### Add Destination Actions and Connect Paths

1. Add the **Send Email** action and the **Create a Ticket in Jira** action (from the Template Library).
2. **Solid Line ("match")** → **Send Email** (for high severity alerts)
3. **Dotted Line ("no match")** → **Create a Ticket in Jira** (for low and medium severity alerts)

---

## Step 8: Configure Destination Actions

You must update both destination actions to reference the data generated by the **Build Summary & HTML** action.

### Configure "Create a Ticket in Jira"

1. Click on the **Create a Ticket in Jira** action.
2. In the Payload editor, update the three fields:
   - **title:** `build_summary_html.output.alert_title`
   - **severity:** `=GET('explode_alerts.individual_item.severity')`
   - **description:** `build_summary_html.output.alert_summary`

### Configure "Send Email"

1. Click on the **Send Email** action.
2. **Recipients:** Enter your email address.
3. **Subject:** Delete any existing text, click **+**, and select: `build_summary_html.output.alert_title`
4. **Body:** Delete any existing text, click **+**, and select: `build_summary_html.output.html_body`

### Verify Payload Integrity

Before running, check both destination actions for syntax errors:

1. **Formula syntax:** When you insert a pill that represents a formula, do not type an equals sign ( `=` ) immediately before or after it. This is not Excel.
2. Check the **Create a Ticket in Jira** payload via the **Plain code** tab.
3. Visually inspect the **Send Email** Subject and Body fields to confirm only clean pills are present.

---

## Step 9: Final Test Run

The story is complete. Time to run it end to end.

1. Click the top action: **Get New Alerts**.
2. Click **Run**.
3. Watch the events flow through each action in sequence and confirm the outputs at each stage match what you expect.


[← Back to Home](README.md)




