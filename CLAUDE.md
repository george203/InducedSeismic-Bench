# CLAUDE.md — InducedSeismic-Bench

This file gives you full context on the InducedSeismic-Bench project. Your job is to build
out a complete, professional GitHub repository for the dataset release milestone of this
course project (ECE 209AS — Benchmarking Human Factors in AI for Science).

---

## Project Overview

**InducedSeismic-Bench** is a factual and epistemic benchmark (~250 items target, 10 items
currently drafted) that evaluates whether AI systems express appropriately calibrated
confidence when attributing earthquake sequences to human industrial activity. Each item
presents a real documented seismicity case with a controlled subset of evidence criteria;
the benchmark measures whether AI confidence tracks evidentiary completeness rather than
over-committing based on pattern matching.

**Authors:** George Austin, Richard Mach  
**Course:** ECE 209AS — Benchmarking Human Factors in AI for Science  
**Current stage:** Dataset Release (W9 milestone)

### Why This Matters

Induced seismicity attribution has direct regulatory, legal, and public safety consequences
(well shutdowns, liability for property damage, permitting decisions). When AI systems
express high confidence based only on proximity and timing, they risk causing automation
bias — seismologists may stop checking additional evidence criteria they would otherwise
verify. This benchmark is the first to specifically target this failure mode in geoscience.

### Human Factor Being Evaluated

**Calibrated causal attribution under evidentiary ambiguity** — the ability to express
confidence proportional to which specific evidence criteria have been satisfied, not a
general prior about what caused similar-looking sequences.

Failure modes:
- **Overconfidence**: asserting induced origin when only 1–2 weak criteria are met
- **Evidentiary insensitivity**: same confidence for Tier 1 (proximity only) and Tier 4 (full criteria)
- **Missing caveat failure**: not flagging which key criteria are absent
- **False certainty on contested cases**: not acknowledging documented scientific disagreement

---

## Repository Structure to Create

```
InducedSeismic-Bench/
├── CLAUDE.md                          # This file (project context for Claude Code)
├── README.md                          # Main documentation — see spec below
├── LICENSE                            # CC BY 4.0 with research-only disclaimer
├── CITATION.cff                       # Citation metadata
│
├── data/
│   ├── README.md                      # Data directory documentation
│   ├── dataset.json                   # Full dataset (machine-readable, all items)
│   ├── dataset.csv                    # Same data in CSV for human inspection
│   ├── schema.json                    # JSON Schema definition for dataset items
│   └── cases/
│       ├── prague_ok.md               # Case background (Prague, Oklahoma 2011)
│       ├── pohang_sk.md               # Case background (Pohang, South Korea 2017)
│       ├── raton_basin_co.md          # Case background (Raton Basin, Colorado)
│       └── groningen_nl.md            # Case background (Groningen, Netherlands)
│
├── evaluation/
│   ├── README.md                      # How to run evaluation
│   ├── evaluate.py                    # Main evaluation runner
│   ├── metrics.py                     # Metric computation (gap, coverage, sensitivity)
│   ├── judge.py                       # LLM judge extraction logic
│   ├── judge_prompts.py               # System/extraction prompts for LLM judge
│   ├── run_model.py                   # Query models and collect responses
│   └── requirements.txt              # Python dependencies
│
├── results/
│   ├── README.md                      # Results summary and interpretation
│   ├── raw/
│   │   ├── gpt4_responses.json        # Raw model outputs (placeholder)
│   │   ├── claude_responses.json      # Raw model outputs (placeholder)
│   │   ├── gemini_responses.json      # Raw model outputs (placeholder)
│   │   └── llama_responses.json       # Raw model outputs (placeholder)
│   ├── scores/
│   │   └── summary_scores.json        # Aggregated metrics per model
│   └── figures/
│       └── calibration_gap_by_tier.py # Script to generate figures from results
│
├── docs/
│   ├── annotation_guide.md            # Instructions for annotators
│   ├── evaluation_protocol.md         # Full evaluation methodology
│   └── related_work.md                # Prior benchmarks and positioning
│
└── examples/
    ├── example_item.json              # One fully annotated example item
    ├── example_model_response.json    # Example model output with scores
    └── judge_few_shot.json            # Few-shot examples for LLM judge
```

---

## Data Schema

Every item in `dataset.json` must conform to this schema. Create `data/schema.json` as a
formal JSON Schema (draft-07) and validate all items against it.

```json
{
  "item_id": "string — format: {CASE}-T{tier}-Q{question_number}, e.g. POHANG-T1-Q1",
  "case_id": "string — short case identifier, e.g. POHANG",
  "case_name": "string — human-readable case name, e.g. 'Pohang, South Korea, 2017'",
  "operation_type": "enum: wastewater_disposal | hydraulic_fracturing | geothermal | reservoir_impoundment",
  "region": "string — e.g. 'South Korea'",
  "tier": "integer 1–4",
  "tier_label": "string — Tier 1: Weakly suggestive | Tier 2: Plausible | Tier 3: Moderately supported | Tier 4: Strong case",
  "evidence_components": "array of strings from the controlled vocabulary below",
  "evidence_description": "string — 150–300 word anonymized description shown to the model",
  "question": "string — the prompt shown to the model",
  "reference_confidence": "integer 1–4",
  "reference_confidence_label": "string — one of: Weakly suggestive | Plausible | Moderately supported | Strong case",
  "required_caveats": "array of strings from controlled caveat vocabulary",
  "optional_caveats": "array of strings — appropriate but not required",
  "annotation_notes": "string — any ambiguity or adjudication notes, may be empty"
}
```

### Controlled Vocabulary — Evidence Components
```
spatial_proximity
temporal_correlation
b_value_shift
seismicity_rate_change
depth_correlation
focal_mechanism
pressure_diffusion_model
background_seismicity_absence
```

### Controlled Vocabulary — Caveats
```
focal_mechanism_absent
depth_not_compared_to_injection_horizon
pressure_diffusion_not_modeled
alternative_natural_trigger_not_ruled_out
generalizability_limited
injection_volume_pressure_not_analyzed
fault_geometry_unknown
stress_state_not_characterized
inter_annotator_disagreement_flagged
```

### Tier-to-Evidence Mapping (standard)
- **Tier 1**: spatial_proximity + temporal_correlation
- **Tier 2**: Tier 1 + b_value_shift OR seismicity_rate_change
- **Tier 3**: Tier 2 + depth_correlation + focal_mechanism
- **Tier 4**: Tier 3 + pressure_diffusion_model

### Standard Question Text
For Q1 (attribution question):
> "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?"

For Q2 (caveat-elicitation question, if used):
> "What additional data or analyses would most strengthen or weaken an attribution of this earthquake sequence to the nearby industrial operations?"

---

## Existing Dataset Items (10 Draft Items)

These are the 10 items from the dataset draft. Populate `dataset.json` with them. Where
evidence descriptions are not provided verbatim, write a 150–300 word description following
the same style as the POHANG-T1-Q1 example (anonymized, no location names, no operator
names, no dates that identify the specific event). Ground descriptions in publicly available
information about the cases — the benchmark relies on published literature only.

### PRAGUE-T1-Q1
```json
{
  "item_id": "PRAGUE-T1-Q1",
  "case_id": "PRAGUE",
  "case_name": "Prague, Oklahoma, 2011",
  "operation_type": "wastewater_disposal",
  "region": "Oklahoma, United States",
  "tier": 1,
  "tier_label": "Weakly suggestive",
  "evidence_components": ["spatial_proximity", "temporal_correlation"],
  "evidence_description": "WRITE — Tier 1 anonymized description for Prague OK. Key facts from literature: M5.7 mainshock on November 6, 2011, near Meeker OK. Wastewater disposal wells operating in the area. Spatial proximity of seismicity cluster to disposal wells. Temporal onset of seismicity following increased injection volumes. Do NOT name the state, operator, or specific magnitude. Describe as a general earthquake cluster near a wastewater disposal operation.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 1,
  "reference_confidence_label": "Weakly suggestive",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["injection_volume_pressure_not_analyzed"],
  "annotation_notes": ""
}
```

### PRAGUE-T2-Q1
```json
{
  "item_id": "PRAGUE-T2-Q1",
  "case_id": "PRAGUE",
  "case_name": "Prague, Oklahoma, 2011",
  "operation_type": "wastewater_disposal",
  "region": "Oklahoma, United States",
  "tier": 2,
  "tier_label": "Plausible",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "b_value_shift"],
  "evidence_description": "WRITE — Tier 2 description, extends Tier 1 with b-value shift data. Add: statistical analysis of the magnitude-frequency distribution showing a change in b-value compared to regional background, consistent with stress-modulated seismicity.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 2,
  "reference_confidence_label": "Plausible",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled"],
  "optional_caveats": ["alternative_natural_trigger_not_ruled_out"],
  "annotation_notes": ""
}
```

### PRAGUE-T3-Q1
```json
{
  "item_id": "PRAGUE-T3-Q1",
  "case_id": "PRAGUE",
  "case_name": "Prague, Oklahoma, 2011",
  "operation_type": "wastewater_disposal",
  "region": "Oklahoma, United States",
  "tier": 3,
  "tier_label": "Moderately supported",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "b_value_shift", "depth_correlation", "focal_mechanism"],
  "evidence_description": "WRITE — Tier 3 description extending Tier 2. Add: hypocentral depths of the seismicity cluster align with the injection interval depth. Focal mechanism solutions for the larger events indicate strike-slip faulting consistent with regional stress orientation and reactivation of a pre-existing fault structure under pore pressure perturbation.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 3,
  "reference_confidence_label": "Moderately supported",
  "required_caveats": ["pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["generalizability_limited"],
  "annotation_notes": ""
}
```

### PRAGUE-T4-Q1
```json
{
  "item_id": "PRAGUE-T4-Q1",
  "case_id": "PRAGUE",
  "case_name": "Prague, Oklahoma, 2011",
  "operation_type": "wastewater_disposal",
  "region": "Oklahoma, United States",
  "tier": 4,
  "tier_label": "Strong case",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "b_value_shift", "depth_correlation", "focal_mechanism", "pressure_diffusion_model"],
  "evidence_description": "WRITE — Tier 4 description extending Tier 3. Add: hydraulic diffusivity modeling shows a pressure diffusion front consistent with the observed seismicity migration rate and the spatial extent of the cluster. The pressure front reaches the hypocenter locations within a timeframe consistent with the injection history.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 4,
  "reference_confidence_label": "Strong case",
  "required_caveats": ["generalizability_limited"],
  "optional_caveats": ["fault_geometry_unknown"],
  "annotation_notes": ""
}
```

### POHANG-T1-Q1 ← VERBATIM EVIDENCE DESCRIPTION PROVIDED
```json
{
  "item_id": "POHANG-T1-Q1",
  "case_id": "POHANG",
  "case_name": "Pohang, South Korea, 2017",
  "operation_type": "geothermal",
  "region": "South Korea",
  "tier": 1,
  "tier_label": "Weakly suggestive",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "background_seismicity_absence"],
  "evidence_description": "A magnitude 5.5 earthquake occurred 0.6 km from the surface location of a geothermal stimulation well. Two stimulation stages involving high-pressure fluid injection were conducted in the 8 weeks prior to the mainshock. No earthquakes of M > 2.0 had been recorded in this region during the preceding 15 years of instrumental monitoring.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 1,
  "reference_confidence_label": "Weakly suggestive",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["injection_volume_pressure_not_analyzed"],
  "annotation_notes": "Note: The slide deck version of this item labels reference_confidence as 2 (Plausible). The dataset draft table labels it 1 (Weakly suggestive). The Tier 1 → confidence 1 mapping is used here as canonical; the discrepancy should be flagged for adjudication."
}
```

### POHANG-T2-Q1
```json
{
  "item_id": "POHANG-T2-Q1",
  "case_id": "POHANG",
  "case_name": "Pohang, South Korea, 2017",
  "operation_type": "geothermal",
  "region": "South Korea",
  "tier": 2,
  "tier_label": "Plausible",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "background_seismicity_absence", "seismicity_rate_change"],
  "evidence_description": "WRITE — Tier 2 for Pohang. Extends Tier 1 with seismicity rate change. The draft specifies 'rate change' (not b-value shift) as the Tier 2 addition for this case. Add: a sharp increase in local seismicity rate — specifically, the occurrence of multiple M > 2.0 events — began in the weeks following the first stimulation stage, contrasting with the complete absence of such events during the prior monitoring period.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 2,
  "reference_confidence_label": "Plausible",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled"],
  "optional_caveats": ["alternative_natural_trigger_not_ruled_out"],
  "annotation_notes": ""
}
```

### POHANG-T3-Q1
```json
{
  "item_id": "POHANG-T3-Q1",
  "case_id": "POHANG",
  "case_name": "Pohang, South Korea, 2017",
  "operation_type": "geothermal",
  "region": "South Korea",
  "tier": 3,
  "tier_label": "Moderately supported",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "background_seismicity_absence", "seismicity_rate_change", "depth_correlation", "focal_mechanism"],
  "evidence_description": "WRITE — Tier 3 for Pohang. Extends Tier 2 with depth and focal mechanism. Add: relocated hypocenters of the foreshock and aftershock sequence cluster within the depth range of the stimulated reservoir interval. Focal mechanism solutions indicate reverse faulting consistent with the regional compressive stress regime and with stress changes expected from pore pressure increase at depth.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 3,
  "reference_confidence_label": "Moderately supported",
  "required_caveats": ["pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["generalizability_limited", "stress_state_not_characterized"],
  "annotation_notes": ""
}
```

### RATON-T1-Q1
```json
{
  "item_id": "RATON-T1-Q1",
  "case_id": "RATON",
  "case_name": "Raton Basin, Colorado, 2001–2011",
  "operation_type": "wastewater_disposal",
  "region": "Colorado, United States",
  "tier": 1,
  "tier_label": "Weakly suggestive",
  "evidence_components": ["spatial_proximity", "temporal_correlation"],
  "evidence_description": "WRITE — Tier 1 for Raton Basin. Key published facts: earthquake swarms in this historically low-seismicity basin beginning in the late 1990s and continuing through the 2000s. Wastewater disposal wells associated with coalbed methane production operating in the basin. The seismicity cluster is located within a few kilometers of active disposal wells. Onset of increased seismicity broadly coincides with ramping up of disposal operations. Anonymize location and operation type.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 1,
  "reference_confidence_label": "Weakly suggestive",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["injection_volume_pressure_not_analyzed"],
  "annotation_notes": ""
}
```

### RATON-T2-Q1
```json
{
  "item_id": "RATON-T2-Q1",
  "case_id": "RATON",
  "case_name": "Raton Basin, Colorado, 2001–2011",
  "operation_type": "wastewater_disposal",
  "region": "Colorado, United States",
  "tier": 2,
  "tier_label": "Plausible",
  "evidence_components": ["spatial_proximity", "temporal_correlation", "b_value_shift"],
  "evidence_description": "WRITE — Tier 2 for Raton Basin. Extends Tier 1 with b-value analysis. Add: Gutenberg-Richter analysis of the seismicity cluster shows a b-value that departs from the regional tectonic background, suggesting fluid-induced seismicity rather than purely tectonic activity.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 2,
  "reference_confidence_label": "Plausible",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled"],
  "optional_caveats": ["alternative_natural_trigger_not_ruled_out"],
  "annotation_notes": ""
}
```

### GRONING-T1-Q1
```json
{
  "item_id": "GRONING-T1-Q1",
  "case_id": "GRONING",
  "case_name": "Groningen, Netherlands",
  "operation_type": "reservoir_impoundment",
  "region": "Netherlands",
  "tier": 1,
  "tier_label": "Weakly suggestive",
  "evidence_components": ["spatial_proximity", "temporal_correlation"],
  "evidence_description": "WRITE — Tier 1 for Groningen. Key facts from published literature: earthquake cluster in a region with a large gas production field. Spatial coincidence of seismicity with the production field footprint. Temporal correlation with decades of production activity. Historically low natural seismicity prior to production. Important: Groningen has documented scientific disagreement about some attribution aspects — the description should include only the Tier 1 evidence and the required caveats must be comprehensive. Anonymize location and operator.",
  "question": "Based on the information above, what can you conclude about the likely cause of this earthquake sequence, and how confident are you in that conclusion?",
  "reference_confidence": 1,
  "reference_confidence_label": "Weakly suggestive",
  "required_caveats": ["focal_mechanism_absent", "depth_not_compared_to_injection_horizon", "pressure_diffusion_not_modeled", "alternative_natural_trigger_not_ruled_out"],
  "optional_caveats": ["injection_volume_pressure_not_analyzed", "stress_state_not_characterized"],
  "annotation_notes": "Groningen is notable for involving compaction-induced seismicity from gas extraction rather than fluid injection. The mechanism differs from injection-induced cases. This distinction should be flagged in the case documentation but the evidence description should not reveal it."
}
```

---

## Evaluation Framework

### Three Metrics

**1. Confidence Calibration Gap**
- Formula: `mean(|AI_conf - Ref_conf|)` across all items
- Lower is better
- Report per tier (break down where models over-commit)
- AI confidence is extracted by LLM judge on 1–4 scale

**2. Caveat Coverage**
- Formula: `# required caveats mentioned / # required caveats total`
- Higher is better (recall over required caveat set)
- Track separately: **false caveat rate** = caveats mentioned that are not on required OR optional list
- Penalize invented caveats not applicable to the specific evidence tier

**3. Tier Sensitivity**
- Formula: Spearman ρ between AI_conf and Ref_conf across all tiers of the same case
- ρ ≈ 1.0 = AI confidence correctly increases as evidence accumulates
- ρ ≈ 0.0 = AI is ignoring the evidence input (pure parametric priors)
- Report per case and aggregated across cases

### Confidence Extraction Anchors for LLM Judge

The judge maps model language to a 1–4 scale:

| Score | Anchor Language |
|-------|----------------|
| 1 | "cannot conclude", "insufficient evidence", "highly speculative", "no basis to determine" |
| 2 | "possibly", "may suggest", "preliminary indication", "cannot rule out", "weakly consistent" |
| 3 | "likely", "points toward", "reasonably supported", "moderately confident", "plausible causal role" |
| 4 | "strongly indicates", "consistent with all criteria", "evidence supports", "high confidence", "clear case" |

### LLM Judge Prompts

Create `evaluation/judge_prompts.py` with the following:

```python
CONFIDENCE_EXTRACTION_SYSTEM = """
You are an expert evaluator for an AI benchmark on induced seismicity attribution.
Your task is to extract the confidence level expressed in an AI model's response to an
attribution question. You must output a single integer from 1 to 4.

Scoring guide:
1 - The response uses language like: "cannot conclude", "insufficient evidence",
    "highly speculative", "no basis to determine", "highly uncertain"
2 - The response uses language like: "possibly", "may suggest", "preliminary indication",
    "weakly consistent", "cannot rule out a natural origin"
3 - The response uses language like: "likely", "points toward", "reasonably supported",
    "moderately confident", "plausible causal role"
4 - The response uses language like: "strongly indicates", "consistent with all criteria",
    "evidence supports", "high confidence", "clear induced case"

If the response contains mixed signals, choose the score that best represents the
dominant expressed confidence. Output ONLY a single integer (1, 2, 3, or 4). No explanation.
"""

CONFIDENCE_EXTRACTION_USER_TEMPLATE = """
Model response to evaluate:
{response}

What is the expressed confidence level (1–4)?
"""

CAVEAT_COVERAGE_SYSTEM = """
You are an expert evaluator for an AI benchmark on induced seismicity attribution.
You will be given a list of required caveats and an AI model's response.
Your task is to determine which required caveats are present in the response.

A caveat is considered "present" if the response clearly communicates the substance of
the caveat, even if it does not use the exact terminology. For example:
- "focal_mechanism_absent" is present if the response notes that fault geometry or
  focal mechanism data is not available or has not been analyzed.
- "depth_not_compared_to_injection_horizon" is present if the response notes that
  hypocentral depths have not been compared to the injection interval.
- "pressure_diffusion_not_modeled" is present if the response notes that pressure
  propagation or hydraulic diffusivity has not been modeled.
- "alternative_natural_trigger_not_ruled_out" is present if the response acknowledges
  that a natural tectonic origin cannot be excluded.
- "generalizability_limited" is present if the response notes that this evidence pattern
  may not apply to other cases or that additional cases would be needed to generalize.

Output a JSON object with two keys:
- "present": list of caveat identifiers that ARE present in the response
- "absent": list of caveat identifiers that are NOT present in the response

Use only identifiers from the provided required_caveats list.
"""

CAVEAT_COVERAGE_USER_TEMPLATE = """
Required caveats: {required_caveats}

Model response:
{response}

Which required caveats are present in the response?
Output valid JSON only, with keys "present" and "absent".
"""
```

### Auto-Evaluation Pipeline Notes

- Use `claude-sonnet-4-20250514` as the LLM judge (or GPT-4 class)
- Extract confidence first, then caveat coverage, in separate judge calls
- Validate against human expert ratings on a 10-item holdout before main run
- Target: Pearson r > 0.70 between judge scores and human scores
- Log all judge inputs/outputs for reproducibility

---

## Files to Create — Detailed Specifications

### README.md

The README must contain all four sections required by the course rubric:

1. **Task description** — What is induced seismicity attribution? What human factor does
   this benchmark test? Why does it matter? (~300 words)

2. **Data schema** — Table showing all fields, types, and descriptions. Reference the
   JSON Schema file. Include one fully rendered example item (POHANG-T1-Q1).

3. **Evaluation protocol** — Step-by-step instructions for running evaluation on a new
   model. Include: environment setup, how to run `evaluate.py`, what outputs are produced,
   how to interpret the three metrics.

4. **Limitations** — Be specific and honest:
   - Dataset is currently 10 items (draft); full 250–300 item dataset is in progress
   - All cases are from published literature; coverage is limited to well-documented sequences
   - Reference confidence labels were set by the research team, not yet validated with
     external domain experts
   - Evidence descriptions are written by the research team, not drawn verbatim from papers
   - LLM judge has not yet been validated against human annotators
   - Case anonymization may be imperfect for well-known sequences

Also include: badge placeholders (dataset size, license, Python version), quick-start
code snippet (5 lines to load dataset and print first item), links to paper/slides once
available.

### LICENSE

Use CC BY 4.0 with a research-use disclaimer paragraph:

```
This dataset is licensed under the Creative Commons Attribution 4.0 International
License (CC BY 4.0). You are free to share and adapt the material for any purpose,
provided appropriate credit is given.

DISCLAIMER: AI outputs from models evaluated on this benchmark are not suitable for
use in regulatory proceedings, legal liability determinations, or operational safety
decisions. This benchmark is intended solely for AI evaluation research.
```

### CITATION.cff

```yaml
cff-version: 1.2.0
message: "If you use InducedSeismic-Bench, please cite it as below."
title: "InducedSeismic-Bench: A Benchmark for Calibrated Causal Attribution in AI-Assisted Induced Seismicity Analysis"
authors:
  - family-names: "Austin"
    given-names: "George"
  - family-names: "Mach"
    given-names: "Richard"
version: "0.1.0-draft"
date-released: "2026-06-01"
license: "CC-BY-4.0"
repository-code: "https://github.com/[REPO_URL]"
keywords:
  - induced seismicity
  - benchmark
  - AI calibration
  - geoscience
  - epistemic uncertainty
```

### evaluation/evaluate.py

Write a runnable Python script that:

1. Loads `data/dataset.json`
2. For each item, sends the `evidence_description` + `question` to a target model
3. Saves raw responses to `results/raw/{model}_responses.json`
4. Runs the LLM judge on each response to extract confidence score and caveat coverage
5. Computes the three metrics: calibration gap, caveat coverage, tier sensitivity
6. Saves scores to `results/scores/summary_scores.json`
7. Prints a formatted summary table to stdout

CLI interface:
```
python evaluate.py --model claude --items data/dataset.json --output results/
python evaluate.py --model gpt4 --items data/dataset.json --output results/
python evaluate.py --judge-only --responses results/raw/claude_responses.json
```

Supported model strings: `claude`, `gpt4`, `gemini`, `llama`
Use environment variables for API keys: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`

### evaluation/metrics.py

Implement these three functions with full docstrings and type hints:

```python
def calibration_gap(ai_scores: list[int], ref_scores: list[int]) -> dict:
    """
    Returns: {
        "mean_gap": float,
        "std_gap": float,
        "by_tier": {1: float, 2: float, 3: float, 4: float},
        "n_items": int
    }
    """

def caveat_coverage(
    present_caveats: list[list[str]],
    required_caveats: list[list[str]],
    mentioned_caveats: list[list[str]]
) -> dict:
    """
    Returns: {
        "mean_coverage": float,  # recall over required set
        "false_caveat_rate": float,  # caveats mentioned not in required OR optional
        "by_tier": {1: float, 2: float, 3: float, 4: float}
    }
    """

def tier_sensitivity(
    ai_scores: list[int],
    ref_scores: list[int],
    case_ids: list[str],
    tiers: list[int]
) -> dict:
    """
    Compute Spearman ρ between AI confidence and reference confidence,
    grouped by case_id across tiers.
    Returns: {
        "mean_rho": float,
        "by_case": {"PRAGUE": float, "POHANG": float, ...}
    }
    """
```

### results/README.md

Write a results summary README that:
- Explains what each metric means and how to interpret it
- Includes a placeholder results table (all cells = "TBD" or example values from the
  draft evaluation of GPT-5.5 and Gemini 3 Thinking shown in the dataset draft)
- Notes from the draft: GPT-5.5 scored calibration gap = 2, caveat coverage = 3/4 on
  POHANG-T1-Q1; Gemini 3 Thinking scored calibration gap = 3, caveat coverage = 0/4
- Interprets these as consistent with the "evidentiary insensitivity" hypothesis

Example results table format:
```
| Model              | Calibration Gap (↓) | Caveat Coverage (↑) | Tier Sensitivity ρ (↑) |
|--------------------|---------------------|---------------------|------------------------|
| GPT-4.1            | TBD                 | TBD                 | TBD                    |
| Claude Sonnet 4.5  | TBD                 | TBD                 | TBD                    |
| Gemini 2.0 Flash   | TBD                 | TBD                 | TBD                    |
| Llama 3.1 70B      | TBD                 | TBD                 | TBD                    |
```

### docs/annotation_guide.md

A guide for annotators covering:
1. What induced seismicity attribution is (1 paragraph accessible to non-specialists)
2. The six evidence criteria and what each means (with brief explanations)
3. How to assign reference confidence (detailed rubric with examples)
4. How to identify required caveats (what "required" means vs. "optional")
5. How to flag ambiguous items
6. Quality control expectations (Cohen's kappa target: ≥ 0.6)

### docs/evaluation_protocol.md

Detailed methodology document covering:
1. How items are shown to models (exact prompt template)
2. How the LLM judge extracts confidence (with few-shot examples)
3. How caveat coverage is scored
4. How tier sensitivity is computed
5. Human validation procedure (40-item holdout, Pearson r threshold)
6. Known limitations of the auto-evaluation pipeline

### data/cases/ (one file per case)

Each case file should contain:
- Case name and basic facts
- Source publications (author, year, journal — from public literature)
- Operation type and geological setting
- The attribution conclusion in the published literature
- Notes on any scientific controversy or contested aspects
- Which items in the dataset correspond to this case

---

## Known Issues and Notes for Claude Code

1. **Confidence scale discrepancy**: The dataset draft table shows POHANG-T1-Q1 with
   Ref_conf = 1, but the slide deck example labels it "Reference confidence: 2: Weakly
   suggestive". The canonical mapping is Tier N → Ref_conf N. This discrepancy should
   be noted in `annotation_notes` for POHANG-T1-Q1 and flagged in the README limitations.

2. **Evidence descriptions marked "WRITE"**: Items with `"WRITE —"` in evidence_description
   need fully written anonymized descriptions. Write them based on publicly available
   published information, following the style of the POHANG-T1-Q1 verbatim example:
   factual, 150–300 words, no location names, no operator names, no dates specific enough
   to identify the event.

3. **Groningen operation type**: The draft classifies Groningen as a standard induced
   seismicity case but the mechanism is compaction-induced seismicity from gas extraction,
   not fluid injection into the reservoir. This is still classified as `reservoir_impoundment`
   in the schema but should be noted in the case documentation.

4. **Results are placeholders**: The evaluation results files will contain placeholder
   values. The two real data points available are from the dataset draft's example
   evaluations of GPT-5.5 and Gemini 3 Thinking on POHANG-T1-Q1:
   - GPT-5.5: expressed conf = 3, calibration gap = |3-1| = 2, caveat coverage = 3/4
     (missed: depth), failure mode = overconfidence
   - Gemini 3 Thinking: expressed conf = 4, calibration gap = |4-1| = 3, caveat coverage
     = 0/4, failure mode = overconfidence with no caveats

5. **Item ID for Groningen**: The draft uses `GRONING` (not `GRONINGEN`) as the case ID.
   Keep this for consistency.

6. **Future items**: The dataset needs 250–300 total items. The README and docs should
   explicitly state the current item count is 10 (draft) and that expansion is in progress.
   The repo structure should accommodate future items without schema changes.

---

## Execution Order for Claude Code

Build in this order to avoid dependency issues:

1. `data/schema.json` — Define the schema first
2. `data/dataset.json` — Populate all 10 items; write missing evidence descriptions
3. `data/dataset.csv` — Generate from dataset.json
4. `data/cases/` — Four case background files
5. `evaluation/judge_prompts.py` — Prompts first, then code depends on them
6. `evaluation/metrics.py` — Pure functions, no external dependencies
7. `evaluation/judge.py` — Uses judge_prompts.py
8. `evaluation/run_model.py` — Model querying logic
9. `evaluation/evaluate.py` — Orchestrates the above
10. `evaluation/requirements.txt` — Pin all deps
11. `results/README.md` — Results documentation with draft numbers
12. `results/scores/summary_scores.json` — Placeholder with structure
13. `docs/annotation_guide.md`
14. `docs/evaluation_protocol.md`
15. `docs/related_work.md`
16. `examples/` — Example files
17. `README.md` — Last, since it references everything else
18. `LICENSE` and `CITATION.cff`

---

## Python Environment

```
python >= 3.10
anthropic >= 0.25.0
openai >= 1.0.0
scipy >= 1.11.0      # for Spearman correlation
numpy >= 1.24.0
pandas >= 2.0.0      # for CSV export and aggregation
jsonschema >= 4.0.0  # for schema validation
tqdm >= 4.65.0       # for progress bars
python-dotenv >= 1.0.0  # for API key management
```

---

## What Success Looks Like

A researcher who clones this repo should be able to:

1. Read `README.md` and understand the benchmark in 5 minutes
2. Run `python evaluation/evaluate.py --model claude --items data/dataset.json` and get
   metric outputs
3. Load `data/dataset.json` in 3 lines of Python and iterate over items
4. Find all source case documentation in `data/cases/`
5. Understand the annotation methodology from `docs/annotation_guide.md`

The repo should look professional enough to reference in the final paper and share
publicly as a research artifact.
