# MVP 1 Architecture

## Scope

MVP 1 contains two manually triggered n8n workflows and a Google Sheets queue. Groq provides the language-model calls. LinkedIn is the publishing destination. A human makes the approval decision in Google Sheets.

## Component view

```text
                  ┌──────────────────────────────┐
                  │ Workflow 1: Content creation │
                  └──────────────┬───────────────┘
                                 │
                      Manual topic configuration
                                 │
                         Groq Content Generator
                                 │
                         Groq Quality Reviewer
                                 │
                          PASS / REWRITE
                           /          \
                        PASS          REWRITE
                         │               │
                  Google Sheets     Groq Rewriter
                    PENDING              │
                         │          Retry / re-review
                         │               │
                         │         REWRITE row at limit
                         ▼
                  Human reviews sheet
                         │
                  Changes to APPROVED
                         │
                  ┌──────▼───────────────────────┐
                  │ Workflow 2: LinkedIn publish │
                  └──────────────┬───────────────┘
                                 │
                       Read APPROVED rows
                                 │
                      Exclude existing URL
                                 │
                        Select one post
                                 │
                         LinkedIn Create Post
                                 │
                  Update sheet: POSTED + URL
```

## Data contract

The queue uses these fields:

- `post_id`: unique row identifier used to match publisher updates.
- `created_date`: timestamp for content creation.
- `publish_slot`: current content slot label.
- `category`, `topic`: content classification.
- `hook`, `post_content`, `hashtags`: generated content fields.
- `source_urls`: source URLs supplied to the generator, or empty for evergreen content.
- `quality_score`: overall reviewer score.
- `status`: workflow state such as `PENDING`, `APPROVED`, `REWRITE`, `POSTED`.
- `scheduled_time`: available template column; current workflow is manually triggered and does not use it for scheduling.
- `linkedin_url`: populated after successful LinkedIn publishing.

## Known boundaries

- RSS Read is disabled in the supplied content workflow export.
- The LLM reviewer is not an independent fact-checking system.
- Human approval is required.
- Publisher selects one post per execution.
- No pre-publish `PUBLISHING` lock or comprehensive failure recovery is present in the supplied publisher export.
