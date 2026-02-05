---
name: criteria-generator
description: Generate an assessment-approach.md file for auditing UK maintained school websites against DfE statutory publication requirements. Use when the user wants to create or update the assessment criteria from the gov.uk guidance page. This skill reads https://www.gov.uk/guidance/what-maintained-schools-must-publish-online and produces a structured assessment approach document that the school-assessor skill uses to perform audits. Run this skill once initially, and again whenever the gov.uk guidance is updated.
---

# Criteria Generator

Generate an `assessment-approach.md` file by reading the DfE's statutory guidance for maintained schools.

## Prerequisites

- Claude Code Chrome integration enabled (`claude --chrome` or `/chrome` command)
- Network access to gov.uk

## Workflow

1. **Navigate to the guidance page**
   - Open https://www.gov.uk/guidance/what-maintained-schools-must-publish-online in Chrome
   - Extract the "Last updated" date from the page metadata

2. **Parse all criteria sections**
   - Extract each numbered section (Admission arrangements through Test/exam results)
   - For each criterion, identify:
     - Section number and title
     - Applicability conditions (school type, phase, employee count, etc.)
     - Requirement level: "must publish" (mandatory) vs "should publish" (recommended)
     - Specific items that must/should be published
     - Any linked statutory guidance or templates

3. **Generate assessment-approach.md**
   - Save to the project directory
   - Follow the output format specification below

## Output Format

Generate `assessment-approach.md` with this structure:

```markdown
# School Website Compliance Assessment Approach

## Metadata

- **Source**: https://www.gov.uk/guidance/what-maintained-schools-must-publish-online
- **Last Updated**: [DATE from gov.uk page]
- **Generated**: [current date/time]

## School Classification Questions

Before beginning an audit, establish the school's profile:

1. **Phase**: Primary / Secondary / All-through
2. **Governance type**: Community / Voluntary-controlled / Foundation / Voluntary-aided
3. **Key stages served**: [KS1] [KS2] [KS4] [KS5] (tick all that apply)
4. **Employee count**: Under 250 / 250 or more
5. **Has sixth form**: Yes / No

## Assessment Criteria

### 1. [Criterion Name]

**Requirement Level**: MANDATORY | RECOMMENDED

**Applies To**:
- [List of school types/conditions, e.g., "All maintained schools", "Foundation and voluntary-aided schools only", "Secondary schools with years 7-13"]

**What to Verify**:
- [Specific item 1]
- [Specific item 2]
- ...

**Evidence Guidance**:
- [Describe what evidence is needed to demonstrate compliance, e.g., "Capture the page(s) showing the full behaviour policy or link to downloadable document"]
- [Note any specific data points that must be visible in evidence]

**Statutory Reference**: [Link to relevant legislation/guidance if mentioned]

---

[Repeat for all criteria...]
```

## Evidence Collection Principles

When auditing, evidence may take various forms depending on how the school presents information:

| Evidence Type | When to Use | What to Capture |
|---------------|-------------|-----------------|
| **Screenshot** | Content displayed directly on webpage | Full page or relevant section showing the required information |
| **External link** | School links to official source (e.g., Ofsted, LA website) | Screenshot showing the link exists, plus verification the link works |
| **Downloaded document** | PDF, Word doc, or other file | The document itself, plus screenshot showing where it's linked from |
| **Multiple pages** | Information spread across several pages | Screenshots of each relevant page, noting navigation path |

**General principles:**
- Capture enough context to prove the information exists and is publicly accessible
- Record URLs for all evidence sources
- For downloadable documents, capture both the download link location and the document content
- Use descriptive filenames following the naming convention below

## Applicability Rules Reference

Use these rules when parsing the guidance:

| Condition | Applies When |
|-----------|--------------|
| All schools | No restrictions |
| Primary schools | Phase is Primary or All-through with primary provision |
| Secondary schools | Phase is Secondary or All-through with secondary provision |
| Schools with KS1 | Key stages include KS1 |
| Schools with KS2 | Key stages include KS2 |
| Schools with KS4 | Key stages include KS4 |
| Schools with KS5 / sixth form | Has sixth form = Yes |
| Foundation/VA schools | Governance type is Foundation or Voluntary-aided |
| Community/VC schools | Governance type is Community or Voluntary-controlled |
| 250+ employees | Employee count is 250 or more |

## Suggested Evidence Naming Convention

For consistent organisation, evidence files can follow this pattern:

```
{section-number:02d}-{criterion-slug}-{descriptor}.{ext}
```

Examples:
- `01-admission-arrangements-policy.png`
- `01-admission-arrangements-document.pdf`
- `07-curriculum-overview.png`
- `13-pe-sport-premium-report.pdf`

The naming convention is a suggestion to help organise evidence files consistently. The assessor should use their judgement on how many screenshots/documents are needed to adequately evidence each criterion.

## Validation

After generating, verify:
- All 20+ criteria sections from the guidance are captured
- Each criterion has clear applicability rules
- Mandatory vs recommended is correctly distinguished
- Evidence guidance describes what needs to be demonstrated (not prescriptive file lists)
- Evidence guidance accommodates different presentation styles (on-page, links, documents)
