# Manning Writing Instructions — Synthesized Guide

This document consolidates all key takeaways and requirements from the Manning writing instruction files into a single reference.

---

## Document Setup (Google Docs)

Use the Manning template file's page setup — do not use default Google Docs settings.

- **Paper size:** Executive (7.25" × 10.5"), not Letter (8.5" × 11")
- **Margins:** Top 1.25", Bottom 1.25", Left 1", Right 1"
- **Orientation:** Portrait
- **Custom Menu:** The Manning template includes a Custom Menu (visible in the toolbar after Extensions > Help) with useful macros: "Check code lines," "Return background color to code," "Insert Callout," "Insert Sidebar," "Insert AI prompt," and "Insert AI response."

---

## Formatting Styles at a Glance

| Element              | Style / Font                          | Notes                                              |
|----------------------|---------------------------------------|----------------------------------------------------|
| Code listings        | Normal text, **7pt Roboto Mono**      | Not Courier New or Source Code                     |
| Inline code          | Normal text, **8pt Courier New**      | No background color; not Consolas or Source Code   |
| Code annotations     | **Arial** font                        | Not Verdana; each annotation gets its own letter   |
| Figure captions      | **Heading 6** style                   | Not Normal text                                    |
| Listing captions     | **Heading 4** style                   | Not Normal text                                    |
| Table captions       | **Heading 5** style, **above** table  | Not below the table; not Normal text               |
| Inline equations     | Same font as body text (Verdana 8pt)  | Use italic; type directly, not as equation objects |
| Block equations      | Times New Roman, 8pt                  | Use Insert > Symbols > Equation editor             |
| Equation captions    | Normal text, Times New Roman, 9pt, bold, dark red berry 2 | For referencing equations       |

---

## Code

- **Code listings** use 7pt Roboto Mono. Never use Courier New, Source Code, or any other font for code blocks.
- **Inline code** uses 8pt Courier New with no background color. Never use Consolas or Source Code.
- **Code annotations** use individual letters (#A, #B, #C, etc.), each on its own line after the code block, with the Arial font. Each annotation should correspond to a marked point in the code.
- **Code with annotations** should be no longer than **55 characters** per line.
- Do not use the "Source Code" font anywhere — it is not the correct Manning style.

---

## Figures and Images

- Figures must always be inserted **"In line"** with text — never as "Move with text" or "Wrap with text."
- Figure captions use **Heading 6** style (not Normal text).
- Caption format follows the pattern: `Figure X.Y Description of the figure`

---

## Tables

- Table captions must be placed **above** the table, not below it.
- Table captions use **Heading 5** style (not Normal text).

---

## Lists

- Bullet lists must use **bullet points** (•), not dashes (–).
- Use the built-in list formatting tools in Google Docs rather than manually typing bullet characters.

---

## Hyperlinks

- Hyperlinks must be set as **actual clickable links**, not as plain or underlined regular text. Use Insert > Link or Ctrl+K to create proper hyperlinks.

---

## Headings

- Do **not** use the Tab feature inside headings. Using tabs in headings causes the natural numbering of section headings to break — the tab shifts the number display, and the complete section headings will not display correctly in the published output.
- Use the built-in heading hierarchy: section numbers follow the pattern `X.Y`, `X.Y.Z`, etc.

---

## Equations

**Inline equations** (within running text):
- Type them directly into the text — do not use the equation object/editor.
- Use italic formatting to distinguish variables from surrounding text.
- Use Verdana, 8pt (matching body text).
- For non-standard mathematical symbols, copy from the Unicode Mathematical Alphanumeric Symbols block: https://www.compart.com/en/unicode/block/U+1D400 or https://en.wikipedia.org/wiki/Mathematical_Alphanumeric_Symbols

**Block equations** (on their own line):
- Use Insert > Symbols > Equation to access the equation editor.
- Use Times New Roman, 8pt.
- Add captions in Normal text style, Times New Roman, 9pt, bold, dark red berry 2 color, following the format `Equation X.Y`.

---

## AI Prompts and Responses

The Manning template provides dedicated formatting for AI content via the Custom Menu:
- **Insert AI prompt** — formats a section as an AI prompt
- **Insert AI response** — formats a section as an AI response

In the published output, prompts and responses appear as distinct, clearly labeled blocks. Keep prompts and responses self-contained and ensure they are realistic and runnable where applicable.

---

## Editing Workflow in Google Docs

### Editing Modes
- **Editing mode** — makes silent (invisible) changes. Use only for non-controversial bulk operations like find-and-replace for style issues.
- **Suggesting mode** — the primary mode for editorial work. Deletions appear as strikethrough, insertions in colored font. Authors can accept or reject each change.
- **Viewing mode** — for previewing or printing the finished file.

### Comments
- **Add comments** by selecting text and clicking the plus (+) icon, or by right-clicking and choosing "Comment."
- **Reply** to comments by clicking on them and using the reply field.
- **Resolve** comments (checkmark icon) when addressed — resolved comments remain in the comment history.
- **Delete** comments only when you want them permanently gone — deleted comments do not appear in history.
- **Mention** others with the @ symbol followed by their name to send email notifications.
- **Assign** comments as action items by mentioning someone and checking the "Assign" box (permissions dependent).

### Find and Replace
- Access via Edit > Find and replace, or Ctrl+F then click the menu icon.
- Choose your editing mode before replacing: use Suggesting if you want the author to review each replacement, or Editing for immediate silent replacement.

### Spell Check
- Access via Tools > Spelling and Grammar.

### Word and Character Count
- Highlight text (or select nothing for the full document), then go to Tools > Word Count to see characters, words, and pages.

---

## Chapter Structure (from Sample Chapter)

A well-structured Manning chapter follows this pattern:

1. **Opening "This chapter covers" block** — a bulleted list of 2–4 key learning objectives at the very top.
2. **Introductory paragraph** — connects to previous chapters and previews what the reader will learn.
3. **Numbered sections** (X.1, X.2, etc.) with descriptive titles and subsections (X.Y.Z).
4. **Code listings** — numbered sequentially (Listing X.1, X.2, etc.) with captions in Heading 4 style and inline annotations (#A, #B, etc.).
5. **Figures** — numbered sequentially (Figure X.1, X.2, etc.) with captions in Heading 6 style.
6. **Callout boxes** — use bold labels like WARNING, TIP, or named sidebars for supplementary information.
7. **Transitions** — each section ends with a sentence bridging to the next topic.
8. **Summary** — a bulleted list of key takeaways at the end of the chapter.

---

## Quick-Reference Checklist

- [ ] Page setup matches Manning template (Executive size, correct margins)
- [ ] Code listings use 7pt Roboto Mono
- [ ] Inline code uses 8pt Courier New, no background color
- [ ] Code annotations use Arial font with individual letters (#A, #B, etc.)
- [ ] Annotated code lines are ≤ 55 characters
- [ ] Figures inserted "In line" (not "Move with text" or "Wrap with text")
- [ ] Figure captions use Heading 6 style
- [ ] Listing captions use Heading 4 style
- [ ] Table captions use Heading 5 style and appear above the table
- [ ] Bullet lists use bullet points, not dashes
- [ ] Hyperlinks are set as actual links, not plain text
- [ ] No tabs used inside headings
- [ ] Inline equations typed directly (not as equation objects), in italic
- [ ] Block equations use the equation editor, Times New Roman 8pt
- [ ] Chapter opens with "This chapter covers" bullet list
- [ ] Chapter ends with a Summary section
