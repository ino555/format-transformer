# format-transformer

A Claude Code skill that learns a data transformation from a single A→B example and applies it to new input — without any configuration or code.

## What it does

Show Claude an example of how data should look before and after (A_example → B_example), then give it new input (A_new). The skill automatically:

1. Extracts the transformation rules from your example
2. Builds an intermediate representation with explicit computed values
3. Validates the output against the example schema
4. Iterates and self-corrects if something doesn't match (up to 5 attempts)

## Supported formats

Works with any combination of:
- SQL query results → CSV (with renamed headers, computed columns)
- Plain text descriptions → JSON (with fixed schema)
- JSON data → Markdown reports
- Spreadsheet tables ↔ structured data
- Any format you can show by example

## Example usage

```
Aşağıdaki A→B örneğini incele ve aynı dönüşümü A_new'e uygula.

A_example:
order_id | unit_price | qty | discount
1001     | 120.00     | 3   | 0.10

B_example:
"Sipariş No","Birim Fiyat","Adet","Net Tutar"
"1001","120,00","3","324,00"

A_new:
order_id | unit_price | qty | discount
2001     | 55.00      | 10  | 0.05
2002     | 340.00     | 2   | 0.20
```

## Key behaviors

- **Fixed schema**: All output records have exactly the same fields as B_example — missing values get `null`, no new keys invented
- **Explicit arithmetic**: Computed columns are calculated step by step (shown in reasoning)
- **Verification loop**: Output is checked against the example before delivery; if it doesn't match, it retries
- **Format fidelity**: Decimal separators, quoting style, Turkish characters — all matched exactly

## Installation

Clone into your Claude Code skills folder:

```bash
# Windows
git clone https://github.com/ino555/format-transformer.git "%USERPROFILE%\.claude\skills\format-transformer"

# macOS / Linux
git clone https://github.com/ino555/format-transformer.git ~/.claude/skills/format-transformer
```

Then restart Claude Code. The skill activates automatically when you show Claude an A→B transformation example.

## How it works internally

The skill uses a 6-step structured workflow:

1. **Parse** — understand input and output formats from the example
2. **Schema extraction** — write out explicit field mappings and transformation rules
3. **Intermediate JSON** — compute all values explicitly before rendering
4. **Validation** — check types, row counts, computed values
5. **Render** — produce final output from validated data
6. **Verification** — compare against B_example, retry if needed

This structured approach means computed columns (like Net Tutar = price × qty × discount) are verified before output, and schema consistency is enforced across all records.
