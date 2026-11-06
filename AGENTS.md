# Repository Guidelines

## Project Overview

This repository is a Vietnamese study guide for the AWS Certified Developer – Associate (DVA-C02) exam, based on Cloud Mentor Pro's `[CMP-DVA] AWS Certified Developer - Associate_v5.pdf`. It is a documentation project, not an application. The PDF covers 16 source sections; this repository currently contains `Section1/` through `Section13/`.

## Architecture & Data Flow

The source PDF and authoring rules in `CLAUDE.md` guide three Markdown deliverables per section:

1. `slide.md` — Vietnamese translation that follows the source slides.
2. `explain_more.md` — Vietnamese practical context clearly distinguished from slide content.
3. `dva_example.md` — English exam-style questions with Vietnamese explanations.

Use source-page spans to establish provenance; repository folder numbers do not necessarily match PDF section numbers. The original PDF sections are: Section 7, CloudFront and AWS Elastic Beanstalk (pages 337–384); Section 8, CloudFormation (385–433); Section 9, messaging (435–477); Section 10, monitoring/audit (478–537); Section 11, Lambda (538–590); Section 12, DynamoDB (611–660); Section 13, API Gateway (663–700). `Section7/` through `Section13/` now contain dedicated materials for those source sections; the former `Section7/` compilation of partial Sections 9–11 has been replaced with source-aligned content.

## Key Directories

- Root: source PDF and `CLAUDE.md`, the detailed content and language contract.
- `Section1/`–`Section13/`: existing section materials; each contains `slide.md`, `explain_more.md`, and `dva_example.md`.
- No application source, test, or build directories are present. `CLAUDE.md` says not to add an `images/` directory or `context.txt` for this workflow.

## Development Commands

There is no application build, run, lint, or test command. Use the documented Poppler PDF tools when preparing or checking content:

```bash
pdfinfo "[CMP-DVA] AWS Certified Developer - Associate_v5.pdf"
pdftotext -f 8 -l 16 "[CMP-DVA] AWS Certified Developer - Associate_v5.pdf" -
pdftoppm -f 8 -l 8 -png "[CMP-DVA] AWS Certified Developer - Associate_v5.pdf" /tmp/dva-page
```

Replace the example page bounds with the pages being documented. `pdftotext` writes extracted text to standard output; `pdftoppm` can render a page when text extraction is insufficient.

## Code Conventions & Common Patterns

There is no application code, so code-specific conventions such as async patterns, dependency injection, or state management do not apply. Follow the Markdown conventions in `CLAUDE.md`:

- Preserve slide hierarchy, bullets, tables, technical meaning, and PDF page references in `slide.md`. Write prose in Vietnamese; keep the technical keywords listed in `CLAUDE.md` in English. Italicize or quote diagram descriptions.
- For each `explain_more.md` topic, explain the mechanism, why it works that way, and alternatives. Include **NÊN (DO)** practices with rationale and comparison, and **KHÔNG NÊN (DON'T)** anti-patterns with consequences and a safer replacement.
- In `dva_example.md`, keep questions and answer choices in English and explanations in Vietnamese. Include the correct answer, reasons other choices are wrong, and vocabulary/context variations where useful.
- Keep source-slide facts distinct from supplemental advice; do not add unsupported claims to the translation. Follow the detailed 72-keyword list and examples in `CLAUDE.md` rather than duplicating that reference here.

## Important Files

- `[CMP-DVA] AWS Certified Developer - Associate_v5.pdf` — source material and page-number authority.
- `CLAUDE.md` — detailed section workflow, language rules, required content formats, and PDF helper commands.
- `SectionN/slide.md`, `SectionN/explain_more.md`, `SectionN/dva_example.md` — current section outputs.

## Runtime/Tooling Preferences

No project runtime, package manager, dependency manifest, or declared application dependencies are present. Use Poppler's `pdfinfo`, `pdftotext`, and `pdftoppm` for PDF work. AWS CLI, Python/Boto3, shell scripts, and package names appearing inside study material are instructional examples, not repository tooling or entry points.

## Testing & QA

No automated test suite, test framework, or coverage configuration is present. Validate content manually against the exact PDF page range and `CLAUDE.md` before finishing:

- `slide.md`: accurate Vietnamese translation, preserved structure and technical keywords, correct page references.
- `explain_more.md`: each topic explains how/why and alternatives, with reasoned NÊN and KHÔNG NÊN guidance and replacement actions.
- `dva_example.md`: English questions/options, Vietnamese explanations, correct-answer rationale, and useful vocabulary/context notes.
- Confirm all three files exist in the section and keep any added material clearly separate from translated slide content.
