# AAAI-27 submission package — rodent EEG vigilance state classification

Converted from the IEEE conference version (`IEEE___XGBoost_Rodent_Sleep.zip`,
`conference.tex` + `IEEEtran.cls`) to the AAAI-27 anonymous submission format
using the official style files in `../AAAI_27/`.

## Contents

| File | Purpose |
| --- | --- |
| `AnonymousSubmission2027.tex` | Main source (AAAI-27 anonymous submission) |
| `AnonymousSubmission2027.pdf` | Compiled paper (6 pages: 5.5 body + references) |
| `ReproducibilityChecklist.tex` | AAAI reproducibility checklist, filled in — compiled **standalone**, not `\input` into the paper |
| `ReproducibilityChecklist.pdf` | Compiled checklist (2 pages), uploaded separately from the paper |
| `references.bib` | Bibliography, reformatted for author–year (natbib + `aaai2027.bst`) |
| `aaai2027.sty`, `aaai2027.bst` | Official AAAI-27 style files (unmodified) |
| `images/` | Figures (feature importance, SHAP, calibration curve) |
| `AnonymousSubmission2027.bbl` | Pre-built bibliography, for submission sites that do not run BibTeX |

## Building

```
pdflatex AnonymousSubmission2027
bibtex   AnonymousSubmission2027
pdflatex AnonymousSubmission2027
pdflatex AnonymousSubmission2027

pdflatex ReproducibilityChecklist    # separate 2-page PDF, run twice
pdflatex ReproducibilityChecklist
```

PDFLaTeX is required — `aaai2027.sty` refuses XeLaTeX and LuaLaTeX.

## What changed relative to the IEEE version

- **Class and preamble.** `IEEEtran` (conference) → `article` + `\usepackage[submission]{aaai2027}`.
  All AAAI "DO NOT CHANGE" preamble lines are preserved verbatim from the template.
  Removed `cite`, `textcomp`, `xcolor`, `adjustbox`, and the duplicate `graphicx`/`amsmath`
  loads; kept `amsmath`/`amssymb`, `enumitem`, `booktabs`, `tikz`, and `pgfplots`.
  None of the AAAI-forbidden packages are used.
- **Title block.** `\IEEEauthorblockN`/`\IEEEauthorblockA` → AAAI `\author`/`\affiliations`.
  The `[submission]` option renders these as "Anonymous submission" automatically and
  swaps the copyright line for the AAAI anonymized-submission notice.
- **Keywords.** The `IEEEkeywords` block was dropped — AAAI papers do not carry keywords.
- **Citations.** Numeric IEEE citations → natbib author–year. `aaai2027.sty` aliases
  `\cite` to `\citep`; narrative citations were changed to `\citet`
  (e.g. "\citet{saevskiy2025_sensors} reported 89% accuracy").
  `\bibliographystyle{IEEEtran}` → `aaai2027.bst` (set automatically by the style file).
- **Bibliography.** Entries normalized for author–year rendering: `and colleagues` /
  `et~al.` in author fields → `and others`; corporate authors brace-protected; en dashes
  and curly quotes replaced with LaTeX equivalents; acronyms brace-protected against
  the style's title-casing.
- **Table.** The hand-built IEEE "TABLE I" (manual `\textsc` header, `\hline`,
  `adjustbox`) → a `booktabs` table with a real `\caption` and `\label`, cross-referenced
  from the text.
- **Sections.** Ordering changed to put **Limitations** before **Conclusion**, per AAAI
  convention. Section numbering is off (`secnumdepth 0`), so IEEE-style `\subsubsection`
  headings were given run-in periods.
- **Text.** Manual `\\` line breaks replaced with real paragraphs; the bold run-in
  "labels" in the feature/architecture descriptions became `itemize` lists;
  non-ASCII characters (non-breaking hyphens, en dashes, curly quotes) replaced with
  LaTeX equivalents so the source compiles with plain PDFLaTeX.
- **Abstract footnote.** The `\footnote` inside the abstract (code repository) was
  inlined — footnotes in the abstract are not permitted by the AAAI template.
- **Reproducibility checklist.** Added and filled in. It is compiled as a **standalone
  document** and submitted as a separate file, which is what AAAI-27 asks for — the
  `\input{ReproducibilityChecklist.tex}` line in the main file is left commented out, as
  in the official template. Uncomment it only if your track wants the checklist inline.

Results, numbers, figures, and claims are unchanged from the IEEE version.

## Points to check before submitting

1. **Precision mismatch (pre-existing).** The abstract, Methodology, Results bullet list,
   and Conclusion all report XGBoost precision as **86.8%**, but Table 1 reports **86.6%**.
   This discrepancy is inherited verbatim from the IEEE source — pick one and make it
   consistent.
2. **Figure 4 caption vs. text.** The text calls it a "SHAP beeswarm"; the supplied
   `images/Shap.png` is a mean-|SHAP| bar plot. Either swap the image or reword the text.
3. **Baseline logistic regression.** The Conclusion cites 54.9% accuracy for the baseline,
   but no logistic-regression row appears in Table 1. Consider adding it.
4. **De-anonymization.** The competition reference (`bdhsc2024case`) was genericized to
   avoid identifying the host institution during review. Restore the full citation in the
   camera-ready version.
5. **Author block.** Fill in real names/affiliations and switch to `CameraReady2027`-style
   options only for the camera-ready version.
