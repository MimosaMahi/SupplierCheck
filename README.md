# SupplierCheck

An n8n workflow that checks garment supplier quotes against the company's purchase request and rules, step by step. Each document gets **Approve**, **Reject** or **Needs a person to check**, with the reason and the quoted text. It never guesses.

## How it works

The workflow is a fixed chain of steps. There is no free-running agent.

```
Upload form (PDF)
   -> Extract text from PDF
   -> Information Extractor (AI: reads fields from the quote)
   -> Check extracted data
   -> Match with the purchase request
   -> Rule engine (plain JavaScript, no AI)
   -> Switch: Approve / Reject / Needs a person to check
   -> Result screen with status and reasons
```

Only one step uses AI: reading the fields from the PDF text. Every decision is made by plain JavaScript that applies the company rules, so the same input always gives the same answer. If a value is missing or unclear, the result is "Needs a person to check".

### Rules applied

**Reject** (any one):
1. Unit price is more than 3% above the maximum in the purchase request
2. Advance payment is more than 20%
3. No late-delivery penalty clause
4. Supplier TIN or BIN is missing
5. Delivery later than 45 days after the PO

**Needs a person to check** (any one, if nothing above applies):
1. Unit price above the maximum, but by no more than 3%
2. Quantity differs from the request by more than 5%
3. Payment terms unclear, or outside 30 to 60 days after delivery
4. Delivery in 31 to 45 days
5. VAT not shown separately at 15%
6. Numbers do not add up (line total, subtotal, VAT or total)
7. Quote valid for less than 30 days

**Approve:** none of the above applies.

The late-delivery penalty must be at least 0.5% of order value per week of delay. A lower penalty is sent to a person to check.

## Files

| File | What it is |
|---|---|
| `SupplierCheck Workflow File.json` | The n8n workflow. Import this file |
| `purchase-request.csv` | The company's purchase request (maximum prices and quantities) |
| `company-rules.md` | The company rules |
| `supplier-pack-01.pdf` to `supplier-pack-10.pdf` | The 10 sample quotes |
| `SupplierCheck Workflow.png` | The n8n workflow image |


## How to run

### Requirements
- n8n (Cloud, or self-hosted)
- An OpenAI API key (the workflow uses the `gpt-5-mini` model)
- Node.js, only if you want to run the rule tests

### Run the workflow
1. In n8n, create a new workflow, open the menu, and choose **Import from File**. Select `SuplierCheck Workflow File.json`.
2. Open the **OpenAI Chat Model** node and select your OpenAI credential.
3. Open the **supplier pack file** (Form Trigger) node. Under Options, check that "Respond When" is set to "Workflow Finishes". If the result screen does not show later, try the other options.
4. Click **Execute workflow**, then open the form's **Test URL**.
5. Upload one supplier PDF and submit. The result screen shows the status, the supplier name and all the reasons.
6. To share a link, activate the workflow and use the Form Trigger's **Production URL**. It works for other people only if your n8n is reachable from the internet. On a local n8n it needs a tunnel such as ngrok.


## Expected results for the 10 sample packs

| Pack | Supplier | Expected | Main reason |
|---|---|---|---|
| 01 | Padma Textile Mills | Approve | No rule triggered |
| 02 | Jamuna Trims | Approve | No rule triggered |
| 03 | Karnaphuli Yarn | Reject | Price 9.05% above maximum |
| 04 | Surma Packaging | Needs a person to check | Price 2.31% above maximum |
| 05 | Meghna Carton | Reject | 40% advance |
| 06 | Teesta Fabrics | Reject | No late-delivery penalty clause |
| 07 | Rupsha Zipper | Needs a person to check | Quantity 17.5% short |
| 08 | Bhairab Thread | Reject | TIN/BIN missing |
| 09 | Dhaleshwari Poly Pack | Needs a person to check | Line total wrong, payment terms unclear |
| 10 | Kushiyara Box | Reject | Delivery 60 days |

**Result from the live workflow:** 10/10 correct 
