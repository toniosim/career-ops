# Mode: pdf — ATS-Optimized PDF Generation

## Resume vs. Full CV

| Feature | Resume (`base-resume.html`) [DEFAULT] | Full CV (`cv-template.html`) |
|---------|---------------------------------------|------------------------------|
| **Goal** | 1-page, high-impact, consistent | Full history, max keywords |
| **Sections** | Summary, Skills (Table), Experience, Edu | Summary, Competencies, Experience, Projects, Edu, Skills |
| **Style** | Professional Blue, Underlined (Antonio Style) | Cyan/Purple, Modern |
| **Generation** | Start from finalized base, tweak summary only | Rebuild from `cv.md` |

## Complete Pipeline

1. Read `templates/base-resume.html` as the source of truth — this is the finalized base resume. Do NOT read `cv.md` for generation.
2. Request JD if not in context (text or URL).
3. Extract 10-15 keywords from the JD.
4. Detect JD language → output language (EN default).
5. Detect company location → paper format:
   - US/Canada → `letter`
   - Rest of the world → `a4`
6. Detect role archetype → adapt framing for the summary only.
7. Rewrite the Professional Summary (3-4 lines) injecting the top JD keywords. Replace the `{{SUMMARY_TEXT}}` placeholder with the tailored summary.
8. Optionally update the role subheader. Replace `{{SUBHEADER}}` with the base title "Software Development Engineer in Test" unless the JD title differs meaningfully (e.g., "QA Engineer", "Automation Engineer"). Only change it if it genuinely improves fit — do not alter for minor wording differences.
9. **Do NOT touch anything else.** No bullet reordering, no keyword injection into experience bullets, no skills table changes. The base resume content is intentionally stable.
10. Read `name` from `config/profile.yml` → normalize to kebab-case lowercase (e.g. "John Doe" → "john-doe") → `{candidate}`.
11. Write the modified HTML to `/tmp/resume-{candidate}-{company}.html`.
12. Execute: `node generate-pdf.mjs /tmp/resume-{candidate}-{company}.html output/resume-{candidate}-{company}-{YYYY-MM-DD}.pdf --format={letter|a4}`.
13. Report: PDF path, page count, top 5 keywords injected into summary.

## Cover Letter Generation

Cover letter PDFs follow the workflow in `apply.md` (Step 5b): select the Senior or Mid template from `cover_letters/`, generate only the company-specific paragraph, present for user approval, then render to PDF.

## ATS Rules (Clean Parsing)

- Single-column layout (no sidebars, no parallel columns).
- Standard headers: "Professional Summary", "Work Experience", "Education", "Skills", "Certifications", "Projects".
- No text in images/SVGs.
- No critical info in PDF headers/footers (ATS ignores them).
- UTF-8, selectable text (not rasterized).
- No nested tables.
- JD keywords injected into Professional Summary only — experience bullets and skills table are not modified.

## PDF Design

- **Fonts**: Arial (Body/Headers), Helvetica.
- **Header**: Name in bold 28pt blue (#1b3a5c) + Subheader with 2px blue underline (#2e75b6).
- **Section Headers**: 11pt bold, blue (#1b3a5c) with 1.5px blue underline.
- **Body**: 10pt grey (#595959), line-height 1.2.
- **Margins**: 0.75in.
- **Background**: Pure white.

## Section Order (Optimized "6-second recruiter scan")

1. Header (Name, Subheader, Contact info).
2. Professional Summary (3-4 lines, keyword-dense).
3. Technical Skills (Categorized table).
4. Work Experience (Reverse chronological).
5. Education.

## Summary Keyword Strategy (Ethical, Truth-based)

Keywords go in the Professional Summary only. Use JD vocabulary to describe real experience — do not invent skills.

Legitimate reformulation examples for the summary:
- JD says "shift-left testing" → summarize as "championing shift-left testing practices embedded in the CI/CD pipeline"
- JD says "test strategy ownership" → summarize as "owning end-to-end test strategy across enterprise platforms"
- JD says "cross-functional collaboration" → summarize as "serving as QA anchor across cross-functional Agile teams"

**NEVER add skills the candidate does not have. Only reformulate real experience with the exact vocabulary of the JD.**

## HTML Base File

Use `templates/base-resume.html` as the base — it contains the fully finalized resume content. Do not use `resume-template.html` (placeholder-based template) for generation.

The only elements you may modify per JD:

| Placeholder | Location | What to replace with |
|-------------|----------|----------------------|
| `{{SUBHEADER}}` | Role subheader line | Job title (default: "Software Development Engineer in Test") |
| `{{SUMMARY_TEXT}}` | Professional Summary div | JD-tailored 3-4 line summary |

All other content — contact info, skills table, experience bullets, education — must remain exactly as in `base-resume.html`.

## Post-generation

Update tracker if the offer is already registered: change PDF status from ❌ to ✅.
