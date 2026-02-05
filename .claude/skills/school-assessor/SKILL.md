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

   Use the Chrome MCP tools to capture and save evidence:

   a) **Start GIF recording for the criterion:**
      ```
      mcp__claude-in-chrome__gif_creator: action=start_recording, tabId={tabId}
      ```

   b) **Take screenshot to capture current state:**
      ```
      mcp__claude-in-chrome__computer: action=screenshot, tabId={tabId}
      ```

   c) **Navigate and scroll through relevant content, taking screenshots at each key point**

   d) **Stop GIF recording:**
      ```
      mcp__claude-in-chrome__gif_creator: action=stop_recording, tabId={tabId}
      ```

   e) **Export GIF to Downloads folder:**
      ```
      mcp__claude-in-chrome__gif_creator: action=export, tabId={tabId}, download=true, filename="{criterion-number}-{criterion-name}.gif"
      ```

   f) **Move the downloaded GIF to evidence folder:**
      ```bash
      mv ~/Downloads/{criterion-number}-{criterion-name}.gif "{school}-evidence/{criterion-folder}/"
      ```

   g) **If there are downloadable documents, get the URL and download them:**
      ```bash
      curl -L -o "{school}-evidence/{criterion-folder}/{document-name}.pdf" "{document-url}"
      ```

   h) **Verify the files were saved:**
      ```bash
      ls -la "{school}-evidence/{criterion-folder}/"
      ```

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

### 3. Evidence Capture Guidelines

**CRITICAL: Use Chrome MCP Tools to Save Evidence Files**

The Chrome MCP provides tools for capturing and saving evidence. Screenshots taken with the `computer` tool are displayed but NOT automatically saved to files. You MUST use the GIF creator to save visual evidence.

**Primary Method: GIF Recording (Recommended)**

For each criterion, use the Chrome MCP GIF creator to record and save evidence:

```
# 1. Start recording before navigating to evidence pages
mcp__claude-in-chrome__gif_creator:
  action: start_recording
  tabId: {your-tab-id}

# 2. Take a screenshot to capture the initial frame
mcp__claude-in-chrome__computer:
  action: screenshot
  tabId: {your-tab-id}

# 3. Navigate, scroll, and screenshot key content
#    (Each screenshot action captures a frame)

# 4. Stop recording
mcp__claude-in-chrome__gif_creator:
  action: stop_recording
  tabId: {your-tab-id}

# 5. Export to Downloads folder with descriptive filename
mcp__claude-in-chrome__gif_creator:
  action: export
  tabId: {your-tab-id}
  download: true
  filename: "01-admission-arrangements.gif"

# 6. Move to evidence folder using Bash
mv ~/Downloads/01-admission-arrangements.gif beatrix-potter-evidence/01-admission-arrangements/
```

**Alternative: Individual Screenshots with System Capture**

If GIF recording is not suitable, use macOS screencapture while Chrome window is visible:

```bash
# Ensure Chrome window is in focus and visible, then:
screencapture -w "{school}-evidence/{criterion-folder}/{filename}.png"
# The -w flag lets you click on the Chrome window to capture it
```

**Downloading Policy Documents**

For linked PDF/DOC files, extract the URL and download:

```bash
# Get document URL from the page, then download
curl -L -o "{school}-evidence/{criterion-folder}/{document-name}.pdf" "{document-url}"
```

**Verification After Each Criterion:**
```bash
ls -la "{school}-evidence/{criterion-folder}/"
# Must show at least one file before proceeding
```

**Workflow for Each Criterion:**

```
For criterion X:
1. Navigate to relevant page(s)
2. Wait for page to load
3. Take screenshot (computer tool, action: screenshot)
4. Save screenshot to file:
   - Use Bash: screencapture or similar tool
   - Filename: {criterion-folder}/{descriptive-name}.png
5. Verify file exists in evidence folder
6. Record evidence in findings
7. Move to next page/criterion
```

**Screenshot best practices:**
- Capture full page or relevant section clearly showing the required information
- Include the URL bar where possible to prove the source
- Save screenshots to the criterion's subfolder IMMEDIATELY after capture
- Use descriptive filenames: `{description}.png` (e.g., `policy-page.png`, `la-application-link.png`)
- VERIFY each file exists before moving to the next criterion

**Document downloads:**
- When a policy document link is found, download it explicitly:
  ```bash
  curl -L -o "{evidence-folder}/{criterion-folder}/{filename}.pdf" "{document-url}"
  ```
- Save linked PDFs/documents to the criterion's subfolder
- Keep original document names or use descriptive names
- Verify the download completed successfully

**For external links (e.g., Ofsted, LA website):**
- Screenshot the page showing the link exists
- Navigate to the external link and screenshot the destination page
- Save BOTH screenshots to prove the link works
- Note in findings that compliance is via external link

**Example evidence folder structure:**
```
beatrix-potter-evidence/
├── 01-admission-arrangements/
│   ├── admissions-page.png
│   ├── oversubscription-criteria.png
│   ├── la-application-link.png
│   └── Beatrix-Potter-Admissions-Policy.pdf
├── 02-behaviour-policy/
│   ├── policies-page-behaviour.png
│   ├── Behaviour-Policy.pdf
│   └── Behaviour-Principles.pdf
├── 06-contact-details/
│   ├── contact-us-page.png
│   └── senco-details.png
├── 07-curriculum/
│   ├── curriculum-overview.png
│   ├── phonics-scheme.png
│   └── ks1-curriculum.png
├── 11-ofsted-reports/
│   ├── ofsted-link.png
│   └── ofsted-report-page.png
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

## Recommendations

[List any actions needed to achieve full compliance, prioritised by:]

### Priority 1: Mandatory requirements not met
[List any]

### Priority 2: Mandatory requirements partially met
[List with specific actions needed]

### Priority 3: Recommended improvements
[List any recommended (non-mandatory) items to improve]

---

## Detailed Findings

### 1. Admission Arrangements

**Status**: [Met / Partially Met / Not Met]

**Requirement**: [Brief summary of what must be published]

**Evidence**:
- [Description of what was found]
- [Location on website]

**Evidence Files**:
- [admissions-page.png](beatrix-potter-evidence/01-admission-arrangements/admissions-page.png)
- [oversubscription-criteria.png](beatrix-potter-evidence/01-admission-arrangements/oversubscription-criteria.png)
- [Beatrix-Potter-Admissions-Policy.pdf](beatrix-potter-evidence/01-admission-arrangements/Beatrix-Potter-Admissions-Policy.pdf)

**Gap**: [Description of any missing items, or "None identified"]

---

### 2. Behaviour Policy

**Status**: [Met / Partially Met / Not Met]

**Requirement**: [Brief summary of what must be published]

**Evidence**:
- [Description of what was found]
- [Location on website]

**Evidence Files**:
- [policies-page-behaviour.png](beatrix-potter-evidence/02-behaviour-policy/policies-page-behaviour.png)
- [Behaviour-Policy.pdf](beatrix-potter-evidence/02-behaviour-policy/Behaviour-Policy.pdf)

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

**Working with Chrome MCP - Evidence Capture Workflow:**

The Chrome MCP tools are prefixed with `mcp__claude-in-chrome__`. Use them as follows:

**Per-Criterion Evidence Workflow:**

```
# 1. START GIF RECORDING for this criterion
mcp__claude-in-chrome__gif_creator:
  action: start_recording
  tabId: {tabId}

# 2. NAVIGATE to the relevant page
mcp__claude-in-chrome__navigate:
  url: "{page-url}"
  tabId: {tabId}

# 3. WAIT for page to load
mcp__claude-in-chrome__computer:
  action: wait
  duration: 2
  tabId: {tabId}

# 4. SCREENSHOT to capture frame (and view content)
mcp__claude-in-chrome__computer:
  action: screenshot
  tabId: {tabId}

# 5. SCROLL and SCREENSHOT if more content exists
mcp__claude-in-chrome__computer:
  action: scroll
  scroll_direction: down
  scroll_amount: 3
  tabId: {tabId}

mcp__claude-in-chrome__computer:
  action: screenshot
  tabId: {tabId}

# 6. STOP GIF RECORDING
mcp__claude-in-chrome__gif_creator:
  action: stop_recording
  tabId: {tabId}

# 7. EXPORT GIF to Downloads
mcp__claude-in-chrome__gif_creator:
  action: export
  tabId: {tabId}
  download: true
  filename: "{criterion-number}-{criterion-name}.gif"

# 8. MOVE GIF to evidence folder (Bash)
mv ~/Downloads/{criterion-number}-{criterion-name}.gif {school}-evidence/{criterion-folder}/

# 9. DOWNLOAD any linked documents (Bash)
curl -L -o "{school}-evidence/{criterion-folder}/{document}.pdf" "{document-url}"

# 10. VERIFY files exist (Bash)
ls -la "{school}-evidence/{criterion-folder}/"
```

**IMPORTANT**: Do NOT proceed to the next criterion until you have:
- Exported and moved at least one GIF to the criterion's evidence subfolder
- Verified the file exists with `ls -la`
- Downloaded any linked policy documents

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
- Screenshot (.png) showing the relevant webpage content
- OR Downloaded document (.pdf/.doc) for policy files
- OR GIF recording (.gif) showing navigation through evidence

### 3. Validation Checklist

**Evidence Files:**
- [ ] Run `ls -la {school}-evidence/*/` to list all evidence files
- [ ] Each applicable criterion folder contains at least 1 file
- [ ] Total file count matches Evidence Index in report
- [ ] No empty subfolders for applicable criteria
- [ ] File sizes are reasonable (screenshots > 10KB, PDFs > 1KB)

**Report Content:**
- [ ] All 20 criteria have been assessed (applicable or marked N/A)
- [ ] Each "Evidence Files" section links to ACTUAL files that exist
- [ ] All linked files can be opened/viewed
- [ ] Gaps are clearly documented for "Not Met" or "Partially Met" items
- [ ] Recommendations are actionable and prioritised

**Compliance Summary:**
- [ ] Applicable criteria shown in main table
- [ ] N/A criteria listed separately with reasons
- [ ] Statistics match the detailed findings

### 4. Fix Missing Evidence

If any evidence files are missing after validation:
1. Re-navigate to the relevant page
2. Capture and SAVE the screenshot explicitly
3. Verify the file was created
4. Update the report if needed
