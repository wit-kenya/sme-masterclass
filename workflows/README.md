# Live build workflows (n8n JSON)

Thirteen workflows, one file each. They match the build reference node for node: same names, same settings, same expressions. No Code nodes. They point at the Airtable base **Live Build Demo** (`appAwRKB9l1c8uisi`).

| File | Workflow |
|---|---|
| W01-sales-create-order.json | W1 · Sales · Create order |
| W02-sales-walk-in-order-form.json | W2 · Sales · Walk-in order form |
| W03-sales-confirm-order.json | W3 · Sales · Confirm order |
| W04-money-record-a-payment.json | W4 · Money · Record a payment |
| W05-money-chase-unpaid-orders.json | W5 · Money · Chase unpaid orders |
| W06-purchases-low-stock-draft-pos.json | W6 · Purchases · Low stock → draft POs |
| W07-purchases-approve-and-send-po.json | W7 · Purchases · Approve and send PO |
| W08-purchases-receive-goods.json | W8 · Purchases · Receive goods |
| W09-inventory-change-a-price.json | W9 · Inventory · Change a price |
| W10-inventory-stock-count.json | W10 · Inventory · Stock count |
| W11-promotions-broadcast.json | W11 · Promotions · Broadcast |
| W12-reports-end-of-day.json | W12 · Reports · End of day |
| W13-ai-order-agent.json | W13 · AI · Order agent |

## 1. Before you import: create the credentials

In n8n, create these credentials and give them **exactly** these names. The workflows refer to them by name.

| Name | Type |
|---|---|
| `Airtable account` | Airtable Personal Access Token API |
| `Gmail account` | Gmail OAuth2 |
| `OpenAI account` | OpenAI |

If a node still shows a credential warning after import, open it and pick the credential from the list.

## 2. Import

For each file: in n8n, click **Create workflow**, then **⋯ → Import from File…**, and choose the file. You can also open a file, copy all of it, and paste it onto an empty canvas.

## 3. Connect the workflows that call each other (3 places)

The links between workflows can't survive an import, so set them once:

| In | Node | Pick |
|---|---|---|
| W1 | **Confirm order** | W3 · Sales · Confirm order. This node is **switched off** on import, so W1 works before W3 exists. Switch it on (select it and press **D**). |
| W2 | **Create order** | W1 · Sales · Create order |
| W13 | **Place order** | W1 · Sales · Create order |

## 4. Change the email address

`you@yourmail.com` appears in **W5** (Email owner), **W6** (Tell the owner), **W12** (Email owner) and **W11** (the Customers filter). Change it to your own.

## 5. Test, then activate

- **W1** and **W3** have test data pinned on their **Input** trigger. Click **Execute workflow**.
- **W8:** click **Listen for test event**, then open `https://YOUR-NAME.app.n8n.cloud/webhook-test/receive-po?po_id=recbaC3XuS0ZKMPbg`.
- **W6:** click **Execute workflow**. It creates PO-2004 and PO-2005.
- Activate **W3**, **W7** and **W8**, then add the three buttons in Airtable (see "Airtable and credentials" in the build reference).

The expected result for every test is in the build reference.

## Notes

- **Each workflow has an "About" note** on the canvas with what it does and how to test it.
- **For attendees using their own copy of the base:** the files point at Live Build Demo. In each Airtable node, choose your own **Base** and **Table** from the lists. All other settings stay the same.
- **W13 uses `gpt-4.1-mini`.** Change the model in the **Model** node if you prefer another.
- **Timezone:** every workflow is set to Africa/Nairobi.
- **Checked:** all 13 files pass n8n's workflow validator with 0 errors. The only warnings are the deliberate "Execute Once" settings, and the switched-off Confirm order node in W1.
