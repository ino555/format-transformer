# format-transformer

A Claude Code skill that learns a data transformation from a single A→B example and applies it to new input — without any configuration or code.

## What it does

Show Claude an example of how data should look before and after (A_example → B_example), then give it new input (A_new). The skill automatically:

1. Extracts the transformation rules from your example
2. Builds an intermediate representation with explicit computed values
3. Validates the output against the example schema
4. Iterates and self-corrects if something does not match (up to 5 attempts)

## Supported formats

Works with any combination of:
- SQL query results → CSV (with renamed headers, computed columns)
- Plain text descriptions → JSON (with fixed schema)
- JSON data → Markdown reports
- Spreadsheet tables ↔ structured data
- Any format you can show by example

## Example usage

```
Look at the A→B example below and apply the same transformation to A_new.

A_example (SQL result):
order_id | unit_price | qty | discount
1001     | 120.00     | 3   | 0.10
1002     | 45.50      | 10  | 0.00

B_example (target CSV):
"Order No","Unit Price","Qty","Discount","Net Total"
"1001","120.00","3","10%","324.00"
"1002","45.50","10","0%","455.00"

A_new:
order_id | unit_price | qty | discount
2001     | 55.00      | 10  | 0.05
2002     | 340.00     | 2   | 0.20
2003     | 15.75      | 20  | 0.00
```

## Key behaviors

- **Fixed schema**: All output records have exactly the same fields as B_example — missing values get `null`, no new keys are invented
- **Explicit arithmetic**: Computed columns are calculated step by step and shown in the reasoning trace
- **Verification loop**: Output is checked against the example before delivery; if it does not match, it retries automatically
- **Format fidelity**: Decimal separators, quoting style, date formats, casing — all matched exactly to the example

## Installation

Clone into your Claude Code skills folder:

```bash
# Windows
git clone https://github.com/ino555/format-transformer.git "%USERPROFILE%\.claude\skills\format-transformer"

# macOS / Linux
git clone https://github.com/ino555/format-transformer.git ~/.claude/skills/format-transformer
```

Then restart Claude Code. The skill activates automatically when you show Claude an A→B transformation example.

## How it works

The skill uses a 6-step structured workflow:

1. **Parse** — understand input and output formats from the example
2. **Schema extraction** — write out explicit field mappings and transformation rules
3. **Intermediate JSON** — compute all values explicitly before rendering
4. **Validation** — check types, row counts, computed values
5. **Render** — produce the final output from validated data
6. **Verification** — compare against B_example, retry if any check fails

This structured approach means computed columns (e.g. Net Total = unit_price × qty × (1 − discount)) are verified before output, and schema consistency is enforced across all records.
