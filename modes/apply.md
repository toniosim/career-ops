# Mode: apply — Live Application Assistant

Interactive mode for when the candidate is filling out an application form in Chrome. Reads what's on screen, loads prior offer context, and generates personalized responses for each form field.

## Requirements

- **Best with visible Playwright**: In visible mode, the candidate sees the browser and Claude can interact with the page.
- **Without Playwright**: the candidate shares a screenshot or pastes questions manually.

## Workflow

```
1. DETECT     → Read active Chrome tab (screenshot/URL/title)
2. IDENTIFY   → Extract company + role from the page
3. SEARCH     → Match against existing reports in reports/
4. LOAD       → Read full report + Section G (if exists)
5. COMPARE    → Does the on-screen role match the evaluated one? If changed → notify
6. ANALYZE    → Identify ALL visible form questions
7. GENERATE   → For each question, generate a personalized response
8. PRESENT    → Display responses formatted for copy-paste
```

## Step 1 — Detect the offer

**With Playwright:** Take a snapshot of the active page. Read title, URL, and visible content.

**Without Playwright:** Ask the candidate to:
- Share a screenshot of the form (Read tool reads images)
- Or paste the form questions as text
- Or provide company + role so we can look it up

## Step 2 — Identify and load context

1. Extract company name and role title from the page
2. Search `reports/` for company name (case-insensitive grep)
3. If match → load the full report
4. If Section G exists → load prior draft answers as a base
5. If NO match → notify and offer to run a quick auto-pipeline

## Step 3 — Detect role changes

If the on-screen role differs from the evaluated one:
- **Notify the candidate**: "The role has changed from [X] to [Y]. Do you want me to re-evaluate or adapt the responses to the new title?"
- **If adapting**: Adjust responses to the new role without re-evaluating
- **If re-evaluating**: Run full A-F evaluation, update report, regenerate Section G
- **Update tracker**: Change role title in applications.md if appropriate

## Step 4 — Analyze form questions

Identify ALL visible questions:
- Free text fields (cover letter, why this role, etc.)
- Dropdowns (how did you hear, work authorization, etc.)
- Yes/No (relocation, visa, etc.)
- Salary fields (range, expectation)
- Upload fields (resume, cover letter PDF)

Classify each question:
- **Already answered in Section G** → adapt the existing response
- **New question** → generate response from the report + cv.md

## Step 5 — Generate responses

For each question, generate a response following:

1. **Report context**: Use proof points from Block B, STAR stories from Block F
2. **Prior Section G**: If a draft answer exists, use it as a base and refine
3. **"I'm choosing you" tone**: Same framework as auto-pipeline
4. **Specificity**: Reference something concrete from the JD visible on screen
5. **career-ops proof point**: Include in "Additional info" if there's a field for it

**Output format:**

```
## Responses for [Company] — [Role]

Based on: Report #NNN | Score: X.X/5 | Archetype: [type]

---

### 1. [Exact form question]
> [Response ready for copy-paste]

### 2. [Next question]
> [Response]

...

---

Notes:
- [Any observations about the role, changes, etc.]
- [Personalization suggestions the candidate should review]
```

## Step 5b — Generate Cover Letter PDF (if the form has a cover letter field)

Per global rule in `_shared.md`: cover letters must use the **same visual design as the CV** (not plain text).

1. **Generate & Present**: Generate the cover letter text following the structure in `_shared.md` rule #0 and present it to the user in the terminal (or as a `.md` block) for review.
2. **User Confirmation (MANDATORY)**: STOP and wait for the user to approve the content. If they request edits, regenerate and repeat step 1.
3. **Build HTML**: Once approved, build an HTML file using `templates/resume-template.html` as the style base — same CSS, same fonts, same color palette — but with a letter layout instead of resume sections:
   - Header: same `.name-header` + `.contact-row` as the resume
   - Body: plain paragraphs with `.summary-text` styling
   - No skills table, no experience bullets
4. **Write & Render**: Write HTML to `/tmp/cover-letter-{candidate}-{company}.html` and run: `node generate-pdf.mjs /tmp/cover-letter-{candidate}-{company}.html output/cover-letter-{candidate}-{company}-{YYYY-MM-DD}.pdf --format={letter|a4}`
5. **Report**: Report the output path so the user knows which file to upload.

## Step 6 — Post-apply (optional)

If the candidate confirms they submitted the application:
1. Update status in `applications.md` from "Evaluated" to "Applied"
2. Update Section G of the report with the final responses
3. Suggest next step: `/career-ops contacto` for LinkedIn outreach

## Scroll handling

If the form has more questions than are visible:
- Ask the candidate to scroll and share another screenshot
- Or paste the remaining questions
- Process in iterations until the entire form is covered
