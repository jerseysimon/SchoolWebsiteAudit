# School Website Audit

Automated compliance auditing of UK maintained school websites against DfE statutory publication requirements, powered by Claude Code skills and Chrome browser automation.

## How It Works

This project uses two **Claude Code skills** that work together:

1. **`/criteria-generator`** -- Reads the [gov.uk guidance](https://www.gov.uk/guidance/what-maintained-schools-must-publish-online) and produces a structured `assessment-approach.md` file listing all 20 DfE criteria, applicability rules, and evidence requirements.

2. **`/school-assessor`** -- Uses Chrome browser automation to navigate a school's website, capture evidence (screenshots as static GIFs + downloaded policy documents), assess compliance against each criterion, and generate a detailed markdown report.

## Prerequisites

### Claude Code

Install Claude Code (Anthropic's CLI tool):

```bash
npm install -g @anthropic-ai/claude-code
```

### Claude in Chrome Extension

The school-assessor skill drives a real Chrome browser to navigate school websites and capture evidence. You need:

1. Install the **Claude in Chrome** browser extension from the [Chrome Web Store](https://chromewebstore.google.com/detail/claude-in-chrome/)
2. Launch Claude Code with Chrome integration enabled:

```bash
claude --chrome
```

> If you forget `--chrome` at launch, you can enable it mid-session with the `/chrome` command.

### Clone This Repo

```bash
git clone https://github.com/jerseysimon/SchoolWebsiteAudit.git
cd SchoolWebsiteAudit
```

The `.claude/skills/` folder contains the skill definitions. Claude Code automatically discovers these when you run from within the project directory.

## Running an Assessment

### Step 1: Generate the Assessment Criteria (One-Time Setup)

The criteria file already exists at `setup/assessment-approach.md`, so you can skip this step unless the gov.uk guidance has been updated.

To regenerate:

```
/criteria-generator
```

This reads the gov.uk page and produces `setup/assessment-approach.md` with all 20 criteria, applicability rules, and evidence guidance.

### Step 2: Run the School Assessment

```
/school-assessor
```

Claude will ask you for:

| Prompt | Example (Beatrix Potter) |
|--------|--------------------------|
| School name | Beatrix Potter School |
| Website URL | https://www.beatrixpotterschool.com |
| School type | Maintained Community Primary School |
| Location | Wandsworth, SW London |

It will then walk through the **school profile questions** to determine which of the 20 criteria apply:

| Question | Example Answer |
|----------|---------------|
| Phase | Primary |
| Governance type | Community |
| Key stages served | KS1, KS2 |
| Employee count | Under 250 |
| Has sixth form | No |
| Receives PE and sport premium | Yes |
| Receives pupil premium | Yes |
| Has school uniform | Yes |

### Step 3: Sit Back

The assessor will then autonomously:

1. Create an evidence folder structure (`{school-slug}-evidence/` with numbered subfolders per criterion)
2. Navigate the school website in Chrome, page by page
3. Capture static GIF screenshots of evidence pages
4. Download policy documents (PDFs, DOCs) linked from the site
5. Assess each criterion as **Met**, **Partially Met**, or **Not Met**
6. Watch for broken links throughout
7. Generate a compliance report at `{school-slug}-compliance-report.md`

## Output Structure

For a school called "Beatrix Potter", the output looks like:

```
beatrix-potter/
├── beatrix-potter-compliance-report.md
└── beatrix-potter-evidence/
    ├── 01-admission-arrangements/
    │   ├── 01-admissions-page.gif
    │   └── Beatrix-Potter-Admissions-Policy-2024-25.pdf
    ├── 02-behaviour-policy/
    │   ├── 02-policies-page-behaviour.gif
    │   ├── Behaviour-Policy.pdf
    │   └── Behaviour-Principles.pdf
    ├── 04-charging-remissions/
    │   ├── 04-policies-page.gif
    │   └── Charging-and-Remissions.doc
    ├── 05-complaints-policy/
    │   ├── 05-policies-page.gif
    │   └── BP-Complaints-Policy-Procedure.pdf
    ├── 06-contact-details/
    │   └── 06-contact-us-page.gif
    ├── 07-curriculum/
    │   ├── 07-curriculum-overview.gif
    │   └── 07-phonics-page.gif
    ├── ...
    ├── 20-financial-benchmarking/
    │   └── 20-financial-benchmarking.gif
    └── broken-links/
```

Each criterion folder contains:
- **Static GIF screenshots** of the relevant webpage sections (not navigation menus or intermediate pages)
- **Downloaded documents** (PDF/DOC/DOCX) for policy files found on the site

Folders are only created for applicable criteria. For example, a primary school won't have a `03-careers-programme/` folder since that's secondary-only.

## The Compliance Report

The generated markdown report includes:

- **School profile** used to determine applicable criteria
- **Compliance summary table** with Met/Partially Met/Not Met status per criterion
- **Non-applicable criteria** listed separately with reasons
- **Broken links** found during the assessment
- **Prioritised recommendations** (mandatory gaps first, then recommended improvements)
- **Detailed findings** per criterion with evidence descriptions, file links, and identified gaps
- **Evidence index** mapping folders to criteria with file counts

## Example: Beatrix Potter School

The `beatrix-potter/` folder contains a completed assessment. Key findings from the February 2026 assessment:

- **14 of 18** applicable criteria fully met
- **4 partially met**: missing RE withdrawal info (curriculum), no salary disclosure statement (financial), no link to compare school performance service (results), weekly hours total not stated (school times)
- **0 not met**
- Notable issues: empty Statutory Policies navigation page, Chair of Governors name inconsistency between pages

## Assessing a Different School

To assess any UK maintained school:

1. Start Claude Code with Chrome from the project directory:
   ```bash
   cd SchoolWebsiteAudit
   claude --chrome
   ```

2. Run the assessor skill:
   ```
   /school-assessor
   ```

3. Answer the school profile questions when prompted

4. The assessment runs autonomously. A new folder is created for each school with its own evidence and report.

Multiple schools can be assessed in the same project -- each gets its own `{school-slug}/` folder.

## Updating the Criteria

If the DfE updates their guidance at gov.uk:

```
/criteria-generator
```

This regenerates `setup/assessment-approach.md` with the latest requirements. Subsequent assessments will use the updated criteria.

## Project Structure

```
SchoolWebsiteAudit/
├── .claude/
│   └── skills/
│       ├── criteria-generator/
│       │   └── SKILL.md          # Skill: reads gov.uk, generates criteria
│       └── school-assessor/
│           └── SKILL.md          # Skill: audits a school website
├── setup/
│   └── assessment-approach.md    # 20 DfE criteria (generated)
├── beatrix-potter/               # Example completed assessment
│   ├── beatrix-potter-compliance-report.md
│   └── beatrix-potter-evidence/
└── README.md
```

## Limitations

- Requires Chrome to be open with the Claude in Chrome extension active
- Cannot navigate to external sites that block automation (some Ofsted pages, for example)
- Evidence capture depends on Chrome MCP screenshot reliability, which can occasionally fail on certain pages
- Downloaded documents are not analysed for content -- the assessor notes when a full document review is needed to confirm compliance
- The assessment is a point-in-time snapshot; school websites change frequently
