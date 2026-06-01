# Data Directory

This directory contains the InducedSeismic-Bench dataset.

## Files

| File | Description |
|------|-------------|
| `schema.json` | JSON Schema (draft-07) defining the structure of each benchmark item |
| `dataset.json` | Full dataset — all benchmark items in machine-readable JSON |
| `dataset.csv` | Same data in CSV format for human inspection and spreadsheet analysis |
| `cases/` | Background documentation for each seismicity case |

## Quick Load

```python
import json

with open("data/dataset.json") as f:
    items = json.load(f)

print(f"{len(items)} items loaded")
for item in items:
    print(item["item_id"], item["tier_label"])
```

## Dataset Statistics (v0.1.0-draft)

| Case | Case ID | Operation Type | Items | Tiers Covered |
|------|---------|----------------|-------|---------------|
| Prague, Oklahoma, 2011 | PRAGUE | wastewater_disposal | 4 | 1–4 |
| Pohang, South Korea, 2017 | POHANG | geothermal | 3 | 1–3 |
| Raton Basin, Colorado, 2001–2011 | RATON | wastewater_disposal | 2 | 1–2 |
| Groningen, Netherlands | GRONING | reservoir_impoundment | 1 | 1 |
| **Total** | | | **10** | |

## Evidence Component Distribution

| Evidence Component | # Items |
|-------------------|---------|
| spatial_proximity | 10 |
| temporal_correlation | 10 |
| background_seismicity_absence | 3 |
| b_value_shift | 5 |
| seismicity_rate_change | 2 |
| depth_correlation | 4 |
| focal_mechanism | 4 |
| pressure_diffusion_model | 1 |

## Schema Validation

To validate all items against the schema:

```bash
python3 -c "
import json, jsonschema
schema = json.load(open('data/schema.json'))
items = json.load(open('data/dataset.json'))
for item in items:
    jsonschema.validate(item, schema)
print(f'All {len(items)} items valid.')
"
```

## Case Background Files

Each file in `data/cases/` documents the real-world case behind the benchmark items:

- `prague_ok.md` — Prague, Oklahoma earthquake sequence (2011)
- `pohang_sk.md` — Pohang, South Korea geothermal-induced earthquake (2017)
- `raton_basin_co.md` — Raton Basin, Colorado earthquake swarms (2001–2011)
- `groningen_nl.md` — Groningen, Netherlands gas-extraction seismicity

These files include source publications, geological setting, attribution conclusions from
the literature, and notes on any scientific controversy. They are for reference only —
the benchmark items use anonymized evidence descriptions that do not name the locations.
