# SupplierCheck

**What I built**
SupplierCheck is an n8n workflow that checks a supplier quote against the company's purchase request and rules. It runs as fixed steps, one after another: upload form, PDF text extraction, an AI extractor that reads the fields (supplier, items, quantities, prices, delivery, payment, penalty, TIN/BIN, validity), data checks, a match with the purchase request, then a rule engine written in plain JavaScript. A Switch sorts the result into **Approve**, **Reject** or **Needs a person to check**, and a final screen shows the status and every reason with the quoted text. The AI only reads the document. All decisions come from code, so the same input always gives the same answer, and anything missing or unclear goes to a person instead of being guessed.

**How I tested it**
- I worked out the correct decision for each of the 10 sample packs by hand, using the company rules.
- I ran the rule engine on the expected extracted values for all 10 packs with a test script. All 10 decisions matched.
- I ran the real PDFs through the live workflow to test the AI extraction as well.

**Results**
- Rule engine on expected values: **10/10**.
- Live workflow on the real PDFs: **10/10**.

**What does not work well**
- The AI extractor can misread a PDF, and a wrong value gives a wrong decision. The prompt makes it copy exact text and return null when unsure, and the code re-checks the arithmetic, but this risk remains.
- "Needs a person to check" shows the result and reasons but does not pause for a reviewer's decision, so it flags a document for a person without waiting for them.
- A penalty below 0.5% per week is sent to a person, because the rules don't say whether it should be rejected.

**Future Work**
- Add a real human step: send "Needs a person to check" cases to a reviewer by email or Slack, wait for their answer, and record it.
- Add a second check on the extraction, such as re-reading the key numbers from the PDF text, to catch misreads.
- Test on more varied quotes, including multi-item quotes, scanned PDFs and different layouts.
- Keep the company rules and purchase request in a sheet or database instead of in the code, so staff can update them without editing JavaScript.
