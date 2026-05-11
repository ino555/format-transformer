---
name: format-transformer
description: >
  Learns a data transformation from an A→B example pair and applies it to new input.
  Use this skill whenever the user provides an example of how data should be converted from one
  format to another — e.g. SQL result → CSV with Turkish headers, plain text → JSON, JSON → report,
  spreadsheet → table, or any other structured format conversion. Even if the user doesn't use the
  word "transform", trigger this skill when they show an example and ask you to do the same thing
  to new data.
---

# Format Transformer

You learn transformation rules from a single A→B example, then apply those exact rules to new A input to produce B output.

## When to use

Any time you see:
- "Do the same to this" + example pair
- "Convert this [format] to [format]" + example
- "Apply this format to my data" + before/after sample
- An A_example + B_example + A_new structure

## Workflow

Work through these steps explicitly — show your reasoning for each one.

### Step 1 — Parse the example pair

Describe what you observe:
- What format is A_example (columns, delimiters, encoding style)?
- What format is B_example?
- How many rows/fields change?

### Step 2 — Extract transformation schema

Write out the mapping rules explicitly:

```
TRANSFORMATION SCHEMA
─────────────────────
Input format:   <describe>
Output format:  <describe>

Field mappings:
  A.field_a → B.field_x  [what changes: rename / type cast / format / compute]
  A.field_b → B.field_y  [direct copy / quoted / decimal format / etc.]
  (derived)  → B.field_z [formula: field_a × field_b × (1 - field_c)]

Structural changes:
  - Added columns: <list>
  - Removed columns: <list>
  - Row-level aggregation: <yes/no and how>

Output format details:
  - Delimiter, quoting style, header casing, decimal separator, date format, etc.
```

**Key rule: Missing fields in the target schema**
If a source item doesn't have a value for a schema field established by the example, use `null` rather than inventing a new field name or omitting the field. Only create new output fields if the A→B example explicitly demonstrates that pattern.

This matters because you're applying a *fixed schema*, not describing each item ad-hoc. Every output record must have **exactly** the same set of keys/columns as the B_example — no more, no less. For example: a Sony headphone missing `batarya_mah` gets `"batarya_mah": null`, not a new `"batarya_saat"` key; an LG TV missing `renk` gets `"renk": null`, not a new `"cozunurluk"` key.

### Step 3 — Intermediate JSON

Before rendering the final output, build an intermediate JSON representation of all output rows. This makes computed values explicit and verifiable:

```json
{
  "_meta": {"source_rows": N, "output_rows": N},
  "rows": [
    {"field_x": "val", "field_y": "val", "field_z": computed_val},
    ...
  ]
}
```

Show your arithmetic for computed fields inline: `field_z = 55.00 × 10 × 0.95 = 522.50`

### Step 4 — Validate the intermediate JSON

Before rendering:
- All required fields present in every record?
- Data types consistent with the example?
- Computed values double-checked?
- Row count matches input?

Fix any issues before continuing.

### Step 5 — Render the final output

Produce the output in the target format, derived directly from the validated intermediate JSON.

### Step 6 — Verification loop (max 5 attempts)

Check your output against the B_example as a template. If **any** check fails: identify the issue, go back and fix it (starting from Step 3 if needed), re-render, and check again. Repeat up to 5 times. Only output when all checks pass, or after 5 attempts (note what remains inconsistent).

**Schema check (most important):**
- Do all output records have exactly the same set of keys/columns as B_example?
- Any extra keys not in the example? → remove them, use null for that value under the correct schema key instead
- Any missing keys that are in the example? → add them with null

**Content check:**
- Value types match (number vs string, integer vs float)?
- Computed values correct? Re-verify the arithmetic.
- No cells left empty where a value or null is expected?

**Format check:**
- Delimiters, quoting, decimal separator, casing, currency symbols — all match B_example exactly?
- Header names match character-for-character (including Turkish characters)?

## Handling edge cases

**Multiple A→B examples:** Extract rules that hold across all of them — use the common pattern, not the first example alone.

**Partial/ambiguous examples:** State your assumptions explicitly before proceeding.

**Unknown format (no A_example):** Ask the user for an example or a description of the target format before continuing.

**Aggregation:** If the example shows grouping (e.g. rep-level summary from row-level data), identify the grouping key and aggregate function from the example, then apply it consistently.

## Output quality principles

- Prefer explicit over implicit: write out computed values, don't leave formulas in cells
- Match the example's style exactly — casing, spacing, number formatting, symbol placement
- Don't add fields the example doesn't show; don't drop fields the example does show
- When in doubt, follow the example rather than guessing
