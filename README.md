# prompt-engineering-reportPrompt Engineering Mini-Study: Few-Shot, Step-by-Step, and Structured Output

Run date: 2026-11-24
Model: gpt-6.1-sol
Reasoning effort: low for all calls
Method: Live calls through the OpenAI-compatible model API; one call per test item and condition. The model, effort setting, and test inputs were held fixed across comparisons. Temperature was not set. The output snippets below are the model's returned message contents, not paraphrases. For the sentiment and extraction experiments, the shared prompt template was sent with each listed input substituted into {text}.

1. Zero-shot vs. few-shot sentiment classification

Task and scoring

Classify six short reviews as exactly one of positive, negative, mixed, or neutral. I marked a match against the expected label set before looking at the model results. Both conditions used the same six test inputs.

Zero-shot prompt

Plain Text


Classify the sentiment of the review as exactly one of: positive, negative, mixed, neutral. Return only the label in lowercase.

Review: {text}



Few-shot prompt (four examples)

Plain Text


Classify the sentiment of the review as exactly one of: positive, negative, mixed, neutral. Return only the label in lowercase.

Examples:
Review: The camera takes crisp photos and the controls are easy to use.
Label: positive
Review: It stopped charging after two days, and the replacement never arrived.
Label: negative
Review: The keyboard feels great, but the trackpad is unreliable.
Label: mixed
Review: The box contains a cable and a quick-start guide.
Label: neutral

Now classify this review:
Review: {text}
Label:



Test inputs and actual outputs

Test review (input)
Expected
Zero-shot raw output
Few-shot raw output
The battery lasts all day and setup was painless.
positive
positive
positive
It froze twice during my first meeting, and support never replied.
negative
negative
negative
The screen is gorgeous, but the battery barely lasts three hours.
mixed
mixed
mixed
The package arrived on Tuesday.
neutral
neutral
neutral
I expected it to be flimsy, but it feels sturdy and works well.
positive
positive
positive
Not terrible, though I wouldn't recommend it.
negative
negative
negative




Result: Zero-shot: 6/6 exact label matches. Few-shot: 6/6 exact label matches.

Analysis: Few-shot examples gave the model concrete demonstrations of the four labels, but they did not improve accuracy on this test set: zero-shot already got every item right. The examples also did not change any of the six outputs, so this experiment cannot show a consistency improvement. This is a small, relatively clear test set; the mildly tricky negation example was still classified correctly in both conditions. Few-shot used a longer prompt without a measurable benefit here, so a harder or more ambiguous dataset would be needed to establish whether it helps.

2. Direct answer vs. step-by-step prompt

Reasoning problem

Both calls used the same multi-step bookstore inventory problem.

Direct prompt (verbatim):

Plain Text


Answer the following word problem. Give the final number and a one-sentence explanation.

A bookstore starts the morning with 4 shelves holding 18 books each. It donates 13 books before lunch. Later, it removes 7 damaged books and receives 2 cartons containing 9 books each. How many books does it have at the end?



Actual raw output:

Plain Text


70 books. The bookstore ends with \(4 \times 18 - 13 - 7 + 2 \times 9 = 70\) books.



Step-by-step prompt (verbatim):

Plain Text


Solve the following word problem. Think step by step, showing only a short calculation for each step, then give the final number.

A bookstore starts the morning with 4 shelves holding 18 books each. It donates 13 books before lunch. Later, it removes 7 damaged books and receives 2 cartons containing 9 books each. How many books does it have at the end?



Actual raw output:

Plain Text


1. Starting books: \(4 \times 18 = 72\)
2. After donation: \(72 - 13 = 59\)
3. After removing damaged books: \(59 - 7 = 52\)
4. Books received: \(2 \times 9 = 18\)
5. Final total: \(52 + 18 = 70\)

**70 books**



Scoring and analysis: Both answers are correct: 4 × 18 − 13 − 7 + 2 × 9 = 70. The direct prompt already returned the correct result and a compact calculation, so asking for step-by-step work did not improve accuracy in this one-pair comparison. It did make the visible arithmetic easier to audit, but the step-by-step response was longer. This is not enough evidence to generalize about reasoning performance; repeated trials and harder problems would be needed. The quoted steps are the model's visible answer, not a claim that they expose all of its internal reasoning.

3. Structured extraction to JSON

Extraction prompt used in both conditions

Plain Text


Extract the named person or business, the transaction/invoice/quote date, the final total amount stated (paid, owed, or quoted), and currency from the text. Do not treat subtotal, tax, discount, or individual item prices as the final total. Normalize dates to YYYY-MM-DD. If date or currency is absent, use an empty string. Output exactly one JSON object with exactly these keys: name (string), date (string), amount (number), currency (string). No extra text.

TEXT:
{text}



The prompt-only condition used that instruction without an API format constraint. The JSON Schema condition used the same prompt and also supplied this strict schema through the API's structured-output feature:

JSON


{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "date": { "type": "string" },
    "amount": { "type": "number" },
    "currency": { "type": "string" }
  },
  "required": ["name", "date", "amount", "currency"],
  "additionalProperties": false
}



Six test inputs and verbatim returned outputs

Input text
Prompt-only raw output
Strict JSON Schema raw output
Receipt for Maya Chen, 14 March 2025. Books $24.00, tax $1.92. TOTAL PAID: $25.92 USD.
{"name":"Maya Chen","date":"2025-03-14","amount":25.92,"currency":"USD"}
{"name":"Maya Chen","date":"2025-03-14","amount":25.92,"currency":"USD"}
Invoice 8841 — Northwind Studio. Issued 2024/07/03. Consulting $1,200.00; VAT $240.00. Balance due: EUR 1,440.00.
{"name":"Northwind Studio","date":"2024-07-03","amount":1440,"currency":"EUR"}
{"name":"Northwind Studio","date":"2024-07-03","amount":1440,"currency":"EUR"}
Paid by Omar Patel on 2025-01-08: lunch, CAD 18.50; tip CAD 3.00. Total charged CAD 21.50.
{"name":"Omar Patel","date":"2025-01-08","amount":21.5,"currency":"CAD"}
{"name":"Omar Patel","date":"2025-01-08","amount":21.5,"currency":"CAD"}
Invoice from Blue Finch Design to Lee. Date: 2 Feb 2024. Services USD 400; discount USD 50; tax USD 28. The amount payable is USD 378.
{"name":"Blue Finch Design","date":"2024-02-02","amount":378,"currency":"USD"}
{"name":"Blue Finch Design","date":"2024-02-02","amount":378,"currency":"USD"}
Receipt: A. Rivera | 12 Nov 2023 | Grand total: 74.20. Currency not printed.
{"name":"A. Rivera","date":"2023-11-12","amount":74.20,"currency":""}
{"name":"A. Rivera","date":"2023-11-12","amount":74.20,"currency":""}
Draft quote for Cedar Labs, prepared April 5, 2025. Estimated quote total: USD 8,750.00. This is an estimate, not a payment.
{"name":"Cedar Labs","date":"2025-04-05","amount":8750,"currency":"USD"}
{"name":"Cedar Labs","date":"2025-04-05","amount":8750,"currency":"USD"}




Results: Prompt-only: 6/6 syntactically valid JSON objects with the expected keys and types; 6/6 exact expected field values. Strict JSON Schema: 6/6 valid objects matching the schema; 6/6 exact expected field values. JSON was parsed and checked mechanically for valid syntax, exactly the required keys, and the expected value types; the extracted values were also compared with the expected records shown below.

Input
Expected extracted values
Maya Chen receipt
{"name":"Maya Chen","date":"2025-03-14","amount":25.92,"currency":"USD"}
Northwind Studio invoice
{"name":"Northwind Studio","date":"2024-07-03","amount":1440,"currency":"EUR"}
Omar Patel receipt
{"name":"Omar Patel","date":"2025-01-08","amount":21.5,"currency":"CAD"}
Blue Finch Design invoice
{"name":"Blue Finch Design","date":"2024-02-02","amount":378,"currency":"USD"}
A. Rivera receipt
{"name":"A. Rivera","date":"2023-11-12","amount":74.2,"currency":""}
Cedar Labs quote
{"name":"Cedar Labs","date":"2025-04-05","amount":8750,"currency":"USD"}




Analysis: The explicit key/type instructions and exact format example in the prompt were enough to produce valid JSON on all six prompt-only inputs. Strict structured output also passed all six and offers a stronger format contract for production use, especially by constraining required keys and rejecting additional ones; however, it did not raise the observed success rate on these deliberately manageable examples. Schema-valid output alone does not guarantee that names, dates, or totals are semantically correct, so field-level validation remains necessary. One iteration note: an initial schema using a nullable numeric amount plus per-field schema descriptions was rejected by the API as Invalid request format and produced no model output. I simplified it to the accepted strict schema above and used test cases with an explicitly stated amount; this report does not count the rejected request as a successful run.

Overall takeaway

On these small tests, clear instructions were sufficient for the easy cases, and none of the three techniques improved measured correctness over its simpler baseline. Few-shot prompting made no difference to these labels; step-by-step prompting exposed a more auditable calculation but did not change the correct answer; and API-enforced JSON Schema added a valuable structural guarantee without improving this already-perfect six-item prompt-only baseline. The results are illustrative, not statistically conclusive: more varied, ambiguous, and repeated test cases would be needed to estimate reliability in real use.

