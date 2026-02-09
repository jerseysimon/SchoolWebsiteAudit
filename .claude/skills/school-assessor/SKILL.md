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

   **CRITICAL: Follow actual navigation links — do NOT guess URLs.**

   School websites use inconsistent URL structures. Always:
   - Use `read_page` with `filter: interactive` to discover actual `href` values
   - Hover over menu items to reveal dropdown submenus, then read the page again
   - Click links using `ref` IDs or the exact URLs from the page
   - Never construct URLs by guessing patterns (e.g., `/key-information/admissions/` may actually be `/Admissions`)

   Example workflow:
   ```
   # 1. Get actual navigation links
   mcp__claude-in-chrome__read_page: tabId={tabId}, filter=interactive

   # 2. Hover to reveal dropdowns
   mcp__claude-in-chrome__computer: action=hover, coordinate=[x,y], tabId={tabId}

   # 3. Read again to get dropdown links
   mcp__claude-in-chrome__read_page: tabId={tabId}, filter=interactive

   # 4. Click using the actual ref or href
   mcp__claude-in-chrome__computer: action=left_click, ref=ref_19, tabId={tabId}
   ```

3. **CAPTURE AND SAVE EVIDENCE (Required for each applicable criterion)**

   **This step is MANDATORY - do not skip it!**

   Use the Chrome MCP tools to capture **static screenshot GIFs** as evidence. Screenshots taken with the `computer` tool are displayed but NOT automatically saved to files — you MUST use the GIF creator to save visual evidence.

   **What to capture:**
   - The specific page or section containing evidence for the criterion
   - Downloaded policy documents (PDF/DOC/DOCX)
   - External link destination pages (e.g., Ofsted report page)
   - Error pages encountered (for broken link reporting)

   **What NOT to capture:**
   - Navigation menus or dropdowns used to find pages
   - Intermediate pages you click through to reach evidence
   - The homepage (unless it contains specific evidence)
   - Pages unrelated to the current criterion

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

   **Troubleshooting: 0 frames captured.** The Chrome MCP GIF creator occasionally captures 0 frames. If this happens, clear (`gif_creator: action=clear`) and retry steps c–f. If it persists, start recording *before* navigation (swap steps a and c) as a fallback — this produces a 2–3 frame GIF instead of a single-frame static image, which is acceptable.

   **If a page requires scrolling**, capture multiple static GIFs of different sections rather than one animated recording. Each GIF should show a specific piece of evidence with a descriptive filename (e.g., `17-ks2-results.gif`, `17-eyfs-results.gif`).

   **For external links (e.g., Ofsted, LA website):** capture a static GIF of the external destination page and note in findings that compliance is via external link.

   Use descriptive filenames that indicate what the image shows (e.g., `06-senco-details.gif`, not `06-screenshot-3.gif`).

   **Do NOT proceed to step 4 until evidence files are confirmed saved!**

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

4. **Assess compliance with detailed analysis**

   **Status Definitions:**
   - **Met**: All required items are published, accessible, current, and complete
   - **Partially Met**: Core requirement present but with quality or completeness gaps
   - **Not Met**: Required items are missing, inaccessible, or severely outdated
   - **N/A**: Criterion does not apply to this school type

   **For EACH criterion, evaluate these quality dimensions:**

   a) **Document Currency**
      - Check for dates on policies and documents
      - Verify documents are from the current or previous academic year
      - Flag any documents older than 2 years as potentially outdated
      - Note if review dates are published and when next review is due
      - For annual publications (e.g., pupil premium, sports premium, results), verify the year matches current/latest data

   b) **Content Completeness**
      - Cross-reference against ALL sub-requirements in assessment-approach.md
      - Don't just check if a page exists — verify each specific element is present
      - For example, "Contact Details" requires: name, address, phone, email, AND SENCO name
      - For "School Hours", both start AND end times should be stated
      - For "Uniform", items AND where to purchase should be included

   c) **Accessibility and Findability**
      - Note how many clicks from homepage to reach the information
      - Check if navigation labels are clear (e.g., "Policies" vs buried in "About Us")
      - Verify documents are downloadable (not just viewable)
      - Note if external links open appropriately
      - Flag if critical information requires excessive scrolling to find

   d) **Document Format Quality**
      - Check if policies are in accessible formats (PDF preferred over DOC for viewing)
      - Note file sizes (very large files may indicate accessibility issues)
      - Verify documents are not password-protected or corrupted
      - Check if scanned documents have searchable text (accessibility requirement)

   e) **Information Consistency**
      - Cross-check key details appear consistently across pages (e.g., school name, contact details)
      - Verify linked information matches (e.g., SENCO name on Contact page matches SEND page)
      - Check that staff names in governance match those in other references

   f) **Link Integrity**
      - Test that document download links work
      - Verify external links (Ofsted, DfE, LA websites) resolve correctly
      - Note any links that redirect unexpectedly

   **Partial Compliance Examples:**
   - Policy exists but is dated 3+ years ago
   - Contact page has address and phone but missing email
   - Curriculum page exists but doesn't cover all year groups
   - Results page has KS2 but missing phonics data
   - School hours shows start time but not finish time
   - Uniform list provided but no information on where to purchase
   - Governors listed but terms of office/appointment dates missing
   - Pupil premium statement exists but is for previous academic year

5. **Record findings with specificity**
   - Document what evidence was found and where
   - Note any gaps or missing items with specific details
   - Reference the ACTUAL evidence files that were saved in step 3
   - Include relative paths to evidence files in the report
   - For "Partially Met" items, specify EXACTLY what is missing or inadequate
   - For "Met" items, still note any minor improvements that could be made

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

[The following two paragraphs must be included verbatim]
This document presents the findings of an automated website compliance review against the Department for Education (DfE) statutory guidance for maintained schools, as published at https://www.gov.uk/guidance/what-maintained-schools-must-publish-online.

**DISCLAIMER** I have created these skills as a layman interpreting the relevant DfE guidance. I am not a lawyer. Please ensure you review the output of the tool, its evidence and come to your own conclusions. As always, AI can make mistakes…

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
- [Number of clicks from homepage]

**Quality Assessment**:
| Dimension | Finding |
|-----------|---------|
| Currency | [e.g., "Policy dated September 2024" or "No date visible"] |
| Completeness | [e.g., "All required elements present" or "Missing: oversubscription criteria"] |
| Findability | [e.g., "2 clicks from homepage via Key Information > Admissions"] |
| Format | [e.g., "PDF, 245KB, accessible"] |

**Evidence Files**:
- [01-admissions-page.gif](evidence/01-admission-arrangements/01-admissions-page.gif)
- [Admissions-Policy.pdf](evidence/01-admission-arrangements/Admissions-Policy.pdf)

**Gap**: [Description of any missing items, or "None identified"]

**Observations**: [Any additional notes about quality, suggestions for improvement, or notable good practice]

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

Broken link indicators are listed in Section 3 "Broken Link Detection" above — refer to that section while navigating.

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
