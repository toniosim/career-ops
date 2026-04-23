# Mode: pdf — ATS-Optimized PDF Generation

## CV vs. Resume Mode (DEFAULT: Resume)

The system now defaults to **Resume Mode** using your custom template.

| Feature | Resume Mode (`resume-template.html`) [DEFAULT] | CV Mode (`cv-template.html`) |
|---------|----------------------------------------------|---------------------------|
| **Goal** | 1-page, high-impact, visual | Full history, Max keywords |
| **Sections** | Summary, Skills (Table), Experience, Edu | Summary, Competencies, Experience, Projects, Edu, Skills |
| **Style** | Professional Blue, Underlined (Antonio Style) | Cyan/Purple, Modern |

### Resume Mode Specifics

- **Template**: Uses `templates/resume-template.html` by default.
- **Skills Mapping**: Merges all skills and competencies into a single `{{SKILLS_HTML}}` block using the table structure:
  ```html
  <tr>
    <td class="skill-label">Category</td>
    <td class="skill-values">Skill 1, Skill 2...</td>
  </tr>
  ```
- **Experience**: Focused on high-signal achievements. The AI naturally prioritizes content to maintain a professional 1-page flow.

## Complete Pipeline

1. Read `cv.md` as the source of truth.
2. Request JD if not in context (text or URL).
3. Extract 15-20 keywords from the JD.
4. Detect JD language → CV language (EN default).
5. Detect company location → paper format:
   - US/Canada → `letter`
   - Rest of the world → `a4`
6. Detect role archetype → adapt framing.
7. Rewrite Professional Summary injecting JD keywords + exit narrative bridge.
8. Select top 3-4 projects most relevant to the offer (if applicable).
9. Reorder experience bullets by JD relevance.
10. Construct competency grid/table from JD requirements.
11. Inject keywords naturally into existing achievements (NEVER invent).
12. Generate full HTML from template + personalized content.
13. Read `name` from `config/profile.yml` → normalize to kebab-case lowercase (e.g. "John Doe" → "john-doe") → `{candidate}`.
14. Write HTML to `/tmp/cv-{candidate}-{company}.html`.
15. Execute: `node generate-pdf.mjs /tmp/cv-{candidate}-{company}.html output/cv-{candidate}-{company}-{YYYY-MM-DD}.pdf --format={letter|a4}`.
16. Report: PDF path, page count, % keyword coverage.

## ATS Rules (Clean Parsing)

- Single-column layout (no sidebars, no parallel columns).
- Standard headers: "Professional Summary", "Work Experience", "Education", "Skills", "Certifications", "Projects".
- No text in images/SVGs.
- No critical info in PDF headers/footers (ATS ignores them).
- UTF-8, selectable text (not rasterized).
- No nested tables.
- Distributed JD keywords: Summary (top 5), first bullet of each role, Skills section.

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

## Keyword Injection Strategy (Ethical, Truth-based)

Legitimate reformulation examples:
- JD says "RAG pipelines" and CV says "LLM workflows with retrieval" → change to "RAG pipeline design and LLM orchestration workflows".
- JD says "MLOps" and CV says "observability, evals, error handling" → change to "MLOps and observability: evals, error handling, cost monitoring".
- JD says "stakeholder management" and CV says "collaborated with team" → change to "stakeholder management across engineering, operations, and business".

**NEVER add skills the candidate does not have. Only reformulate real experience with the exact vocabulary of the JD.**

## HTML Template

Use the template in `resume-template.html`. Replace placeholders `{{...}}` with personalized content:

| Placeholder | Content |
|-------------|-----------|
| `{{LANG}}` | `en` |
| `{{NAME}}` | (from profile.yml) |
| `{{EMAIL}}` | (from profile.yml) |
| `{{LINKEDIN_URL}}` | [from profile.yml] |
| `{{LINKEDIN_DISPLAY}}` | [from profile.yml] |
| `{{LOCATION}}` | [from profile.yml] |
| `{{SUMMARY_TEXT}}` | Tailored summary with keywords |
| `{{SKILLS_HTML}}` | Table rows with categorized skills |
| `{{EXPERIENCE}}` | HTML for each job with reordered bullets |
| `{{EDUCATION}}` | HTML for education |

## Post-generation

Update tracker if the offer is already registered: change PDF status from ❌ to ✅.
