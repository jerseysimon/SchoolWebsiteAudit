---
name: school-assessor
description: Perform a compliance assessment of a UK maintained school website against DfE statutory publication requirements. Uses the assessment-approach.md criteria and Chrome integration to audit the school's website, capturing evidence and producing a detailed compliance report.
---

# School Website Assessor

Assess a maintained school's website for compliance with DfE statutory publication requirements.

## Prerequisites

- Claude Code Chrome integration enabled (`claude --chrome` or `/chrome` command)
- `assessment-approach.md` file exists in the project directory (run `/criteria-generator` if missing)
- School website URL

## Workflow

### 1. Pre-flight Checks

Before starting an assessment:

1. **Verify assessment-approach.md exists**
   - Read the file from the project directory
   - If missing, prompt user: "No assessment-approach.md found. Run `/criteria-generator` to create it."

2. **Check criteria currency**
   - Extract the "Last Updated" date from assessment-approach.md metadata
   - Compare against current gov.uk guidance date (24 October 2024 as of this skill creation)
   - If outdated, prompt user: "The assessment criteria may be outdated (last updated: [DATE]). Consider running `/criteria-generator` to refresh."

3. **Gather school information**
   - Ask the user for:
     - School name
     - School website URL
     - School type (e.g., "Maintained Community Primary School")
     - Location (e.g., "Wandsworth, SW London")

4. **Establish school profile**
   - Work through the School Classification Questions from assessment-approach.md:
     - Phase: Primary / Secondary / All-through
     - Governance type: Community / Voluntary-controlled / Foundation / Voluntary-aided
     - Key stages served: KS1 / KS2 / KS4 / KS5
     - Employee count: Under 250 / 250 or more
     - Has sixth form: Yes / No
     - Receives PE and sport premium: Yes / No
     - Receives pupil premium: Yes / No
     - Has school uniform requirement: Yes / No
   - Use this profile to determine which criteria are applicable

5. **Create evidence folder structure**
   - Create main folder: `{school-name-slug}-evidence/` (e.g., `beatrix-potter-evidence/`)
   - Create numbered subfolders for each applicable criterion:
     ```
     beatrix-potter-evidence/
     ├── 01-admission-arrangements/
     ├── 02-behaviour-policy/
     ├── 04-charging-remissions/
     ├── 05-complaints-policy/
     ├── 06-contact-details/
     ├── 07-curriculum/
     ├── ...
     ```
   - Only create subfolders for criteria that apply to the school (skip N/A criteria)

### 2. Assessment Process

For each of the 20 criteria in assessment-approach.md:

1. **Determine applicability**
   - Check the "Applies To" rules against the school profile
   - If not applicable, mark as N/A with reason and move to next criterion

2. **Navigate the school website**
   - Use Chrome to browse the school's website
   - Look for the required information as specified in "What to Verify"
   - Check common locations: Policies page, Key Information, About Us, dedicated topic pages

3. **CAPTURE AND SAVE EVIDENCE (Required for each applicable criterion)**

   **This step is MANDATORY - do not skip it!**

   Use the Chrome MCP tools to capture **static screenshot GIFs** as evidence. Only capture pages that directly show evidence for the criterion being assessed - do NOT capture intermediate navigation pages, menus, or pages you pass through to reach the evidence.

   **Per-criterion evidence capture workflow:**

   a) **Navigate to the page containing the evidence:**
      ```
      mcp__claude-in-chrome__navigate: url="{evidence-page-url}", tabId={tabId}
      ```

   b) **Wait for page to load:**
      ```
      mcp__claude-in-chrome__computer: action=wait, duration=2, tabId={tabId}
      ```

   c) **Start GIF recording (single frame capture):**
      ```
      mcp__claude-in-chrome__gif_creator: action=start_recording, tabId={tabId}
      ```

   d) **Take ONE screenshot of the relevant content:**
      ```
      mcp__claude-in-chrome__computer: action=screenshot, tabId={tabId}
      ```

   e) **Stop recording immediately (single frame = static image):**
      ```
      mcp__claude-in-chrome__gif_creator: action=stop_recording, tabId={tabId}
      ```

   f) **Export as static GIF:**
      ```
      mcp__claude-in-chrome__gif_creator: action=export, tabId={tabId}, download=true, filename="{criterion-number}-{descriptive-name}.gif"
      ```

   g) **Move the downloaded GIF to evidence folder:**
      ```bash
      mv ~/Downloads/{criterion-number}-{descriptive-name}.gif "{school}-evidence/{criterion-folder}/"
      ```

   h) **If there are downloadable documents, get the URL and download them:**
      ```bash
      curl -L -o "{school}-evidence/{criterion-folder}/{document-name}.pdf" "{document-url}"
      ```

   i) **Verify the files were saved:**
      ```bash
      ls -la "{school}-evidence/{criterion-folder}/"
      ```

   **If a page requires scrolling**, capture multiple static GIFs of different sections rather than one animated recording. Each GIF should show a specific piece of evidence with a descriptive filename (e.g., `17-ks2-results.gif`, `17-eyfs-results.gif`).

   **Do NOT proceed to step 4 until evidence files are confirmed saved!**

4. **Assess compliance**
   - **Met**: All required items are published and accessible
   - **Partially Met**: Some required items are present but gaps exist
   - **Not Met**: Required items are missing or inaccessible
   - **N/A**: Criterion does not apply to this school type

5. **Record findings**
   - Document what evidence was found and where
   - Note any gaps or missing items
   - Reference the ACTUAL evidence files that were saved in step 3
   - Include relative paths to evidence files in the report

### 3. Broken Link Detection

**CRITICAL: Monitor for broken links throughout the entire assessment.**

While navigating the website for any reason, watch for pages that indicate a broken or dead link. This includes:

- HTTP error codes displayed on the page (e.g., 404, 403, 500, 502, 503)
- Pages containing text such as: "page not found", "sorry", "link broken", "no longer available", "this page doesn't exist", "error", "not found", "moved or deleted", "expired"
- Pages that load but show only a generic error template
- Links that redirect to an unexpected page (e.g., back to the homepage)

**When a broken link is detected:**
1. Note the URL that was requested
2. Note where the link was found (which page linked to it)
3. Note the error message or HTTP code displayed
4. Capture a static GIF of the error page as evidence
5. Save to `{school}-evidence/broken-links/`

**These must be reported regardless of whether they relate to any assessment criterion.** They are reported in a dedicated section of the compliance report (see Output Format below).

### 4. Evidence Capture Guidelines

**CRITICAL: Use Chrome MCP Tools to Save Evidence Files**

The Chrome MCP provides tools for capturing and saving evidence. Screenshots taken with the `computer` tool are displayed but NOT automatically saved to files. You MUST use the GIF creator to save visual evidence as **static (single-frame) GIFs**.

**What to Capture (DO):**
- The specific page or section that contains evidence for the criterion
- Downloaded policy documents (PDF/DOC/DOCX)
- External link destination pages (e.g., Ofsted report page)
- Error pages encountered (for broken link reporting)

**What NOT to Capture:**
- Navigation menus or dropdowns used to find pages
- Intermediate pages you click through to reach evidence
- The homepage (unless it contains specific evidence)
- Pages unrelated to the current criterion

**Static GIF Capture Method:**

For each evidence screenshot, use this compact workflow:

```
# 1. Navigate to evidence page
mcp__claude-in-chrome__navigate:
  url: "{evidence-page-url}"
  tabId: {tabId}

# 2. Wait for load
mcp__claude-in-chrome__computer:
  action: wait
  duration: 2
  tabId: {tabId}

# 3. Start recording
mcp__claude-in-chrome__gif_creator:
  action: start_recording
  tabId: {tabId}

# 4. Take ONE screenshot
mcp__claude-in-chrome__computer:
  action: screenshot
  tabId: {tabId}

# 5. Stop recording immediately
mcp__claude-in-chrome__gif_creator:
  action: stop_recording
  tabId: {tabId}

# 6. Export static GIF
mcp__claude-in-chrome__gif_creator:
  action: export
  tabId: {tabId}
  download: true
  filename: "{criterion-number}-{descriptive-name}.gif"

# 7. Move to evidence folder
mv ~/Downloads/{criterion-number}-{descriptive-name}.gif {school}-evidence/{criterion-folder}/
```

**For pages with long content requiring multiple captures:**

Scroll to each relevant section separately and capture individual static GIFs with descriptive names:
- `07-curriculum-overview.gif` (top of curriculum page)
- `07-phonics-scheme.gif` (scrolled to phonics section)
- `07-re-withdrawal.gif` (scrolled to RE section)

**Downloading Policy Documents:**

For linked PDF/DOC files, extract the URL and download:

```bash
curl -L -o "{school}-evidence/{criterion-folder}/{document-name}.pdf" "{document-url}"
```

**Verification After Each Criterion:**
```bash
ls -la "{school}-evidence/{criterion-folder}/"
# Must show at least one file before proceeding
```

**Screenshot best practices:**
- Only capture pages that directly show evidence for the criterion
- Use descriptive filenames that indicate what the image shows (e.g., `06-senco-details.gif`, not `06-screenshot-3.gif`)
- VERIFY each file exists before moving to the next criterion

**For external links (e.g., Ofsted, LA website):**
- Capture a static GIF of the external destination page
- Note in findings that compliance is via external link

**Example evidence folder structure:**
```
beatrix-potter-evidence/
├── 01-admission-arrangements/
│   ├── 01-admissions-page.gif
│   ├── 01-oversubscription-criteria.gif
│   └── Beatrix-Potter-Admissions-Policy.pdf
├── 02-behaviour-policy/
│   ├── 02-behaviour-policy-link.gif
│   └── Behaviour-Policy.pdf
├── 06-contact-details/
│   ├── 06-footer-contact.gif
│   └── 06-senco-details.gif
├── 07-curriculum/
│   ├── 07-curriculum-overview.gif
│   ├── 07-phonics-scheme.gif
│   └── 07-re-withdrawal.gif
├── 11-ofsted-reports/
│   └── 11-ofsted-report-page.gif
├── broken-links/
│   ├── broken-pe-sport-premium-404.gif
│   └── broken-old-newsletter-link.gif
└── ...
```

## Output Format

Generate `{school-name-slug}-compliance-report.md` with this structure:

```markdown
# Website Compliance Report: [School Name]

## Document Information

- **School**: [Full school name]
- **School Type**: [e.g., Maintained Community Primary School]
- **Location**: [e.g., Wandsworth, SW London]
- **Website**: [URL]
- **Assessment Date**: [Date and time]

This document presents the findings of a website compliance review against the Department for Education (DfE) statutory guidance for maintained schools, as published at https://www.gov.uk/guidance/what-maintained-schools-must-publish-online.

## School Profile

The following profile was used to determine which requirements apply:

| Question | Answer |
|----------|--------|
| Phase | [Primary / Secondary / All-through] |
| Governance type | [Community / Voluntary-controlled / Foundation / Voluntary-aided] |
| Key stages served | [e.g., KS1, KS2] |
| Employee count | [Under 250 / 250 or more] |
| Has sixth form | [Yes / No] |
| Receives PE and sport premium | [Yes / No] |
| Receives pupil premium | [Yes / No] |
| Has school uniform | [Yes / No] |

## Compliance Summary

### Applicable Criteria

| # | Criterion | Mandatory | Status |
|---|-----------|-----------|--------|
| 1 | Admission Arrangements | Yes | [Met/Partially Met/Not Met] |
| 2 | Behaviour Policy | Yes | [Status] |
| 4 | Charging and Remissions | Yes | [Status] |
| 5 | Complaints Policy | Yes | [Status] |
| 6 | Contact Details | Yes | [Status] |
| 7 | Curriculum | Yes | [Status] |
| ... | [Only include criteria that apply to this school] | ... | ... |

*Only criteria applicable to this school are shown above

### Summary Statistics

- **Met**: X of Y applicable criteria
- **Partially Met**: X criteria
- **Not Met**: X criteria

### Non-Applicable Criteria

The following criteria do not apply to this school:

| # | Criterion | Reason |
|---|-----------|--------|
| 3 | Careers Programme | Primary school (Years 7-13 only) |
| 12 | Pay Gap Reporting | Under 250 employees |
| ... | ... | ... |

---

## Broken Links

During the assessment, the following broken or problematic links were identified on the school website. These are reported regardless of whether they relate to a specific compliance criterion.

| Link URL | Linked From | Error | Evidence |
|----------|-------------|-------|----------|
| [URL that was broken] | [Page where the link was found] | [e.g., 404 Not Found] | [broken-link-evidence.gif](evidence-path) |
| ... | ... | ... | ... |

*If no broken links were found, state: "No broken links were identified during this assessment."*

---

## Recommendations

[List any actions needed to achieve full compliance, prioritised by:]

### Priority 1: Mandatory requirements not met
[List any]

### Priority 2: Mandatory requirements partially met
[List with specific actions needed]

### Priority 3: Recommended improvements
[List any recommended (non-mandatory) items to improve]

### Broken Links to Fix
[List any broken links found, with the source page and suggested fix]

---

## Detailed Findings

### 1. Admission Arrangements

**Status**: [Met / Partially Met / Not Met]

**Requirement**: [Brief summary of what must be published]

**Evidence**:
- [Description of what was found]
- [Location on website]

**Evidence Files**:
- [01-admissions-page.gif](evidence/01-admission-arrangements/01-admissions-page.gif)
- [Admissions-Policy.pdf](evidence/01-admission-arrangements/Admissions-Policy.pdf)

**Gap**: [Description of any missing items, or "None identified"]

---

[Repeat structure for all applicable criteria...]

---

## Evidence Index

All evidence files are organised in criterion-specific subfolders:

| Folder | Criterion | Files |
|--------|-----------|-------|
| `01-admission-arrangements/` | #1 Admission Arrangements | 3 files |
| `02-behaviour-policy/` | #2 Behaviour Policy | 2 files |
| `broken-links/` | Broken Links | X files |
| ... | ... | ... |

Total evidence files: [X]
```

## Assessment Tips

**Common page locations to check:**
- Homepage footer (contact details, accessibility statement)
- "Key Information" or "About Us" sections
- "Policies" or "Statutory Policies" page
- "Governance" or "Governors" page
- "Curriculum" section with subject pages
- "SEND" or "Special Educational Needs" page
- "Results" or "Performance" page
- "Admissions" page
- Quick links menu

**Common gaps to watch for:**
- Missing £100k+ salary statement (even if "none", must be stated)
- Missing paper copies availability statement
- Weekly hours total not explicitly calculated
- Outdated documents (check dates on policies)
- Broken links to external services
- SENCo contact details missing from mainstream schools

**Broken link indicators to watch for:**
- HTTP status codes in page content: 404, 403, 500, 502, 503
- Text phrases: "page not found", "sorry", "not available", "broken", "doesn't exist", "moved", "deleted", "expired", "no longer"
- Generic error page templates from the web host or CMS
- Pages that unexpectedly redirect to the homepage

## Validation

**CRITICAL: Verify Evidence Files Exist**

After completing the assessment, run the following validation steps:

### 1. Verify Evidence Folder Structure
```bash
# List all evidence subfolders and their contents
find {school}-evidence -type f -name "*.png" -o -name "*.pdf" -o -name "*.gif" | head -50

# Count files per subfolder
for dir in {school}-evidence/*/; do
  echo "$dir: $(ls -1 "$dir" 2>/dev/null | wc -l) files"
done
```

### 2. Verify Minimum Evidence Per Criterion
Each applicable criterion subfolder MUST contain at least ONE file:
- Static GIF screenshot (.gif) showing the relevant webpage content
- OR Downloaded document (.pdf/.doc) for policy files

### 3. Validation Checklist

**Evidence Files:**
- [ ] Run `ls -la {school}-evidence/*/` to list all evidence files
- [ ] Each applicable criterion folder contains at least 1 file
- [ ] Total file count matches Evidence Index in report
- [ ] No empty subfolders for applicable criteria
- [ ] File sizes are reasonable (screenshots > 10KB, PDFs > 1KB)
- [ ] All GIFs are static (single frame) - no animated recordings

**Report Content:**
- [ ] All 20 criteria have been assessed (applicable or marked N/A)
- [ ] Each "Evidence Files" section links to ACTUAL files that exist
- [ ] All linked files can be opened/viewed
- [ ] Gaps are clearly documented for "Not Met" or "Partially Met" items
- [ ] Recommendations are actionable and prioritised
- [ ] Broken Links section is present (even if no broken links found)

**Compliance Summary:**
- [ ] Applicable criteria shown in main table
- [ ] N/A criteria listed separately with reasons
- [ ] Statistics match the detailed findings

### 4. Fix Missing Evidence

If any evidence files are missing after validation:
1. Re-navigate to the relevant page
2. Capture a static GIF screenshot
3. Verify the file was created
4. Update the report if needed
