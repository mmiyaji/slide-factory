# Slide Factory — Project Context and Handoff

> This document summarizes the design discussion that led to Slide Factory. It is a curated handoff, **not a verbatim transcript**. Treat proposals as proposals rather than settled implementation decisions.

## Background

The project targets PowerPoint presentations for system requirements, architecture, basic design, technical proposals, and stakeholder decision-making. The primary users author materials with **Codex and Claude Code**.

A recurring problem with AI-generated presentations is that diagrams created with draw.io or other graphics tools are embedded as flattened images. They can look polished, but individual elements are difficult to edit, reuse, or adjust in PowerPoint. Generic slide generation also tends to oversimplify content or produce attractive yet weakly structured pages.

The goal is to achieve the information density, logical structure, emphasis, and audience-oriented communication commonly found in strong consulting deliverables **without sacrificing native editability**.

## Project identity

- Repository and skill name: **slide-factory**
- Working description: **An agent-native design system for professional, editable presentations**
- Primary output: **PPTX**
- Initial agent targets: **Codex and Claude Code**
- Primary language/use case: Japanese, information-dense business and IT system documentation; English should also work
- Public/open-source project; no separate nickname or brand has been selected
- Microsoft PowerPoint is a product trademark; use the product name descriptively, not as the project's brand

## Core product concept

**Design once. Adapt to any master. Build natively.**

Separate three assets:

1. **Master Template Registry:** user-provided PPTX masters/themes controlling slide dimensions, fonts, palette, background, logos, margins, header/footer, and protected regions.
2. **Slide Pattern Library:** curated examples of layouts and information structures (e.g., comparison, architecture, flow, summary). Reference styling should not be copied blindly.
3. **Native Component Library:** reusable implementations of tables, shapes, connectors, charts, and diagram nodes as editable PowerPoint objects.

Given content, an audience, and an optional master template, the system should select an appropriate pattern, adapt it to the master, render editable PPTX, and validate both content and appearance.

## Design principles and requirements

- **Communication first:** assertion-based titles, clear key messages, supporting evidence, coherent deck-level storyline, appropriate emphasis and visual hierarchy.
- **High information density without unnecessary cognitive load:** group information, align comparison criteria, use tables and diagrams where appropriate, and avoid dropping important requirements merely to fit a layout.
- **Audience-aware:** consider executives, business owners, reviewers, and engineering teams; communicate what the reader needs to understand or decide.
- **Native editability first:** text, tables, comparison matrices, architecture diagrams, flows, and connectors should be editable PPTX objects wherever practical. Do not flatten an entire diagram or table into PNG/SVG merely for visual polish.
- **Master-driven appearance:** target-master colors, typography, margins, logos, and layout rules take precedence over a reference slide's decoration. Preserve the reference's useful information structure.
- **Japanese typography:** handle long Japanese headings, dense tables, line wrapping, font fallback, and readable annotation.
- **Source fidelity:** preserve traceability to requirements and distinguish facts, assumptions, recommendations, and unresolved decisions. Do not fabricate technical claims or evaluation results.
- **Reproducibility:** keep a structured source of truth in version control so outputs can be regenerated and reviewed.
- **Licensing:** track reference URLs, provenance, and usage conditions. Public availability or free download does not imply redistribution rights. Do not commit third-party templates or customer materials to the public repository without explicit permission.

## Proposed architecture (subject to Codex evaluation)

Input documents (Markdown, PPTX, PDF, and optionally spreadsheets)
→ **Storyline/Slide Planner**
→ **Slide IR** (structured intermediate representation, e.g., JSON/YAML with schema validation)
→ **Pattern Matching + Master Design Tokens**
→ **Layout Engine + Native Components**
→ **PPTX Renderer**
→ **Quality Gates**
→ editable PPTX and diagnostics.

Potential technologies discussed:
- **PptxGenJS** for generating editable PPTX text, shapes, tables, charts, and lines.
- **Zod or JSON Schema** for validating Slide IR and pattern metadata.
- OOXML parsing for inspecting source PPTX and output structure.
- PowerPoint automation on Windows or careful OOXML manipulation as possible options when strict preservation of an existing master is required.
- Render-and-inspect loop for visual QA; validate against actual PowerPoint rendering where practical.

Important caveat: PptxGenJS is well suited to creating presentations but is not a complete editor/importer for arbitrary existing PPTX masters. Distinguish:
- **Design-conformance mode:** extract master tokens and reproduce appearance in newly generated slides.
- **Strict master-inheritance mode:** preserve the source PPTX master/layout structures, potentially requiring a different technical approach.

Do not promise full fidelity or automatic round-trip preservation of manual edits without testing.

## Initial slide pattern taxonomy

Prioritize reusable structures over collecting decorative templates:

- Executive summary / conclusion + evidence
- Issue / evidence / recommendation
- Comparison matrix and scoring
- As-Is / To-Be
- System architecture and layered architecture
- Data flow and integration diagrams
- Business process / swimlane
- Sequence / processing flow
- Issue–cause–countermeasure
- Risk and decision matrix
- Roadmap and migration phases
- RACI / responsibility allocation
- Decisions, open questions, and constraints

Start with a smaller native component set (roughly 8 core families), even if the reference catalog contains dozens of variants.

## Pattern ingestion and metadata

Potential workflow:
1. Import PPTX/PDF or reference URLs.
2. Record origin, author, license, attribution, and allowed use.
3. Render thumbnails and inspect layout/text/object structure.
4. Classify slide purpose, information density, hierarchy, and visual pattern.
5. Normalize away brand-specific colors and decorations.
6. Deduplicate similar patterns and register a reusable description.
7. Implement selected patterns independently as editable native components.

Example pattern fields:
- id, name, category, language
- suitable_for / audience / communication_intent
- structure and constraints
- density, maximum options/criteria
- emphasis strategy
- editable element coverage
- source URL and license
- component implementation status

A reference-only entry is not automatically an asset that can be bundled or redistributed.

## Reference discovery: candidate starting points

The following were discussed as **candidates**, not yet audited or imported. Verify URLs, licensing, download availability, native object structure, and suitability before adding anything:

Japanese-friendly sources:
- プレゼンデザイン: https://ppt.design4u.jp/templates/
- ビズデザ PowerPoint materials: https://www.tridge.work/2023/09/18/pp_materials_dl/
- Bizroute templates: https://exia.co.jp/bizroute/ppt-template.html
- Bizroute system architecture templates: https://exia.co.jp/bizroute/system_configuration.html
- Microsoft PowerPoint templates: https://powerpoint.cloud.microsoft/create/ja/presentation-templates/
- SlidesGratis: https://slidesgratis.org/ja/
- ピアのパソコン学習ガイド: https://pier-pc.com/guide/powerpoint/template/

Consulting reference decks/patterns:
- Slideworks consulting collections: https://slideworks.io/resources/750-real-consulting-presentations-from-mckinsey-deloitte-ey-and-more
- Slideworks BCG examples: https://slideworks.io/resources/54-real-bcg-presentations
- 2Slides examples: https://2slides.com/examples/consulting-presentations
- StrategyU consulting templates: https://strategyu.co/strategy-consulting-powerpoint-templates/
- Slideration: https://budding.site/slideration/en/
- Deckary starter pack: https://deckary.com/template/consulting-starter-pack

For Japanese consulting/report design references, also investigate public government-commissioned reports, e.g. METI: https://www.meti.go.jp/topic/data/e90622aj.html

**Do not assume any of these are unrestricted or suitable for inclusion in a public repository.** Prefer recording links and original, independently implemented patterns.

## Quality assurance

Use independent checks rather than relying on the generating LLM's self-rating:

1. **Structural:** OOXML object types, slide bounds, editable text/shape/table/chart coverage, image-only diagram detection, master compliance.
2. **Visual:** render each slide and check clipping, overlap, alignment, font legibility, whitespace, hierarchy, and consistent style. Intentional overlaps must be supported.
3. **Semantic:** compare against source requirements, check storyline continuity, evidence, technical connections, assumptions, and unsupported claims.

Treat serious factual/technical errors as blockers, regardless of overall visual quality. A useful practical KPI is **human correction time**, alongside native editability rate, error count, and template compliance.

## Suggested CLI / skill interactions (illustrative, not implemented)

```sh
slide-factory master import corporate.pptx --id corporate
slide-factory patterns import examples.pptx
slide-factory patterns search --type comparison --density high
slide-factory build requirements.md --master corporate --output requirements.pptx
slide-factory validate requirements.pptx --render
```

Natural-language interactions should support importing masters, collecting and finding patterns, generating new decks, adapting existing slides, and reviewing quality.

## Suggested implementation phases

1. **Foundation:** define minimal Slide IR, master tokens, PPTX renderer, and Japanese typography test cases.
2. **Pattern library:** import and classify a small set of rights-cleared or reference-only examples; build search/preview and deduplication.
3. **Native components:** implement common comparison, architecture, process, and summary layouts.
4. **Agent integration:** expose a reusable Skill to Codex and Claude Code with reproducible CLI scripts.
5. **Quality loop:** structure checks, rendering, semantic/visual review, and selective repair.

A useful initial PoC is a ~10-slide Japanese system-requirements presentation using one user-provided master, with editable diagrams and tables.

## Decisions versus open questions

**Agreed direction**
- Use the name `slide-factory`.
- Keep the initial public repository minimal and concept-driven.
- Delegate actual Skill architecture and coding to Codex.
- Prioritize Japanese consulting-style slides, native editability, master adaptation, and reusable pattern collection.

**Open / to investigate**
- Exact Skill packaging and cross-agent compatibility.
- Renderer choice and handling of strict PPTX master inheritance.
- Slide IR and pattern schemas.
- How to incorporate manual PowerPoint edits into regeneration workflows.
- Licensing policy for pattern ingestion and any redistributed assets.
- Which freely available sources actually provide editable native objects.
- Automated quality thresholds and rendering environment.
- Project license (MIT was suggested but not yet selected).

## Codex handoff request

Read this document and `README.md` first. Preserve the product intent while challenging unverified assumptions. Propose an architecture, milestones, risks, and the smallest working PoC before building a large template library. Prioritize an editable Japanese PPTX with master-driven styling and objectively testable quality. Do not ingest or publish third-party files without license review.
