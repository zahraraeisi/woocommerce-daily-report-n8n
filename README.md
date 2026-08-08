**Repository Description:**

> An n8n workflow that collects the previous day's WooCommerce orders, calculates order metrics, groups statuses, and sends a daily report to Telegram.

و این هم **README.md** کامل و تمیز:

````markdown
# WooCommerce Daily Report Automation with n8n

A practical n8n workflow that automatically collects the previous day's WooCommerce orders, calculates key order metrics, groups order statuses, and sends a daily report to Telegram.

This workflow is designed for online store managers who want a simple daily overview without manually checking WooCommerce every day.

## What This Workflow Does

The workflow:

- Retrieves the previous day's WooCommerce orders
- Counts the total number of orders
- Calculates the total order value
- Groups orders by status
- Builds a readable daily report
- Sends the report automatically to Telegram
- Runs every day at a scheduled time

## Workflow Logic

```text
Schedule Trigger
      ↓
WooCommerce Orders
      ↓
Convert Order Total to Number
      ↓
 ┌─────────────────────┐
 │                     │
 ↓                     ↓
Order Metrics      Group by Status
 │                     │
 ↓                     ↓
Summarize           Aggregate
 │                     │
 └──────────┬──────────┘
            ↓
           Merge
            ↓
      Build Report
            ↓
         Telegram
````

## Requirements

You will need:

* WordPress
* WooCommerce
* n8n
* WooCommerce REST API credentials
* Telegram Bot
* Telegram Chat ID

## WooCommerce REST API

In your WordPress dashboard go to:

```text
WooCommerce → Settings → Advanced → REST API
```

Create a new API key and give it the required read permission.

Then connect the credentials to the WooCommerce node inside n8n.

Never publish your Consumer Key or Consumer Secret.

## Get Previous Day Orders

The WooCommerce node retrieves orders from the previous day.

### After

```javascript
{{ $today.minus({ days: 1 }).toISO() }}
```

### Before

```javascript
{{ $today.toISO() }}
```

## Convert Order Total to Number

WooCommerce may return the order total as a string.

Before calculating the total value, convert it to a number:

```javascript
{{ Number($json.total) }}
```

## Order Metrics

The workflow calculates:

* Total number of orders
* Total order value

These values are generated using the Summarize node.

## Group Orders by Status

Orders are also grouped by WooCommerce status.

Examples:

```text
processing
completed
failed
cancelled
```

The workflow then aggregates these results before generating the final Telegram message.

## Report Template

The final report is generated dynamically:

```javascript
{{
`📊 گزارش روزانه فروشگاه

📅 تاریخ گزارش: ${new Intl.DateTimeFormat('fa-IR-u-ca-persian', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  timeZone: 'Asia/Tehran'
}).format($now.minus({ days: 1 }).toJSDate())}

تعداد سفارش‌ها: ${$json.count_id}
مجموع مبلغ سفارش‌ها: ${Number($json.sum_total_number).toLocaleString('fa-IR')} تومان

وضعیت سفارش‌ها:
${$json.statuses
  .filter(item => item.status !== 'pws-in-stock')
  .map(item => {
    const labels = {
      processing: 'در حال پردازش',
      failed: 'ناموفق',
      cancelled: 'لغوشده',
      completed: 'تکمیل‌شده'
    };

    return `• ${labels[item.status] || item.status}: ${item.count_id}`;
  })
  .join('\n')}`
}}
```

## Telegram Setup

Create a Telegram bot using BotFather.

Then:

1. Get your Telegram Bot Token
2. Add the Telegram credential in n8n
3. Get your Chat ID
4. Select your Telegram credential in the Telegram node

Never publish your Bot Token or private Chat ID.

## Scheduling

The workflow uses the n8n Schedule Trigger.

You can choose any daily execution time.

The example workflow uses:

```text
Timezone: Asia/Tehran
```

Make sure your n8n timezone is configured correctly.

## Important: Order Value vs Realized Revenue

This workflow calculates the total value of the retrieved orders.

Depending on your WooCommerce setup, this may include orders with statuses such as:

* processing
* failed
* cancelled
* completed

Therefore, this number should be interpreted as **total order value**, not necessarily finalized or realized revenue.

If you want to calculate actual revenue, you should first define which order statuses count as completed sales and filter the orders accordingly.

## Currency

The example report displays the amount in:

```text
Toman
```

The workflow itself does not perform currency conversion.

You can change the report text based on your WooCommerce store currency.

## How to Import

1. Download the workflow JSON file from this repository.
2. Open your n8n dashboard.
3. Create a new workflow.
4. Choose **Import from File**.
5. Import the JSON file.
6. Add your own WooCommerce credentials.
7. Add your own Telegram credentials.
8. Configure your Telegram Chat ID.
9. Test the workflow.
10. Activate it.

## Security

Before sharing or publishing any n8n workflow, make sure you remove:

* Customer information
* Pinned execution data
* API keys
* WooCommerce credentials
* Telegram Bot Tokens
* Chat IDs
* Authorization headers
* Private URLs
* Sensitive test data

Never publish production customer data inside a public GitHub repository.

## YouTube Tutorial

I published a complete step-by-step tutorial showing how this workflow works and how to build it with n8n.

Watch the tutorial:

[https://youtu.be/3QVv3fYSgSs](https://youtu.be/3QVv3fYSgSs)

## Possible Extensions

This workflow can be extended with features such as:

* Daily revenue reports
* Product sales reports
* Low-stock alerts
* Top-selling products
* Failed order alerts
* Weekly reports
* Monthly reports
* Google Sheets integration
* Email reports
* AI-generated report summaries
* Advanced WooCommerce analytics

## About Danira

Danira builds practical AI agents, automations, workflows, and operational systems for WordPress, WooCommerce, and online businesses.

## Author

Zahra Raeisi

## License

MIT License

If this workflow was useful, consider starring the repository.

