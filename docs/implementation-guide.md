# MVP 1 Implementation Guide — Local n8n Community Edition

This guide follows the supplied n8n exports. It describes the current implementation and calls out settings that each developer must supply. It does not claim that the project has been independently reproduced.

## 1. Prepare the local environment

1. Install and run n8n Community Edition locally using the official n8n installation instructions.
2. Open your local n8n editor in a browser.
3. Keep the instance local for this MVP. Do not expose it publicly without understanding n8n security and deployment requirements.
4. Create a Groq API key in your own Groq account. Store it only in n8n Credentials; never paste it into workflow fields or GitHub.
5. Prepare a Google account and a LinkedIn account that you control.

## 2. Create the Google Sheets queue

Create a spreadsheet and a tab named `Content_Queue`. The provided CSV contains the columns used by the workflow:

`post_id, created_date, publish_slot, category, topic, hook, post_content, hashtags, source_urls, quality_score, status, scheduled_time, linkedin_url`

The content-generation workflow appends a passing post with status `PENDING`. The publisher searches for rows with status `APPROVED`, then updates the matching `post_id` to `POSTED` and writes a LinkedIn URL.

Human approval is a manual step: inspect the post in the sheet and change its status from `PENDING` to `APPROVED` only when you are satisfied with it. Do not approve unverified claims.

## 3. Import the workflows

In n8n, use the workflow menu's **Import from File** option and import:

- `workflows/01-content-generation-quality-review.json`
- `workflows/02-approved-linkedin-publisher.json`

Both exports are intentionally inactive. Keep them inactive until credentials and mappings are configured.

## 4. Configure credentials

### Groq

Create an n8n Groq credential using your own API key. Assign it to each Groq Chat Model node in the content workflow. The export uses these model identifiers:

- Content Generator: `openai/gpt-oss-20b`
- AI Quality Reviewer: `openai/gpt-oss-120b`
- AI Post Rewriter: `openai/gpt-oss-120b`

Model availability, limits and names can change. Confirm the models are available to your Groq account and adjust them if needed. If you change models, re-test quality and token usage.

### Google Sheets

Create/connect your own Google Sheets OAuth2 credential in n8n. Assign it to the Google Sheets nodes in both workflows.

In each Google Sheets node, select your own spreadsheet and the `Content_Queue` tab. The sanitized export includes a placeholder instead of the original spreadsheet ID; do not leave the placeholder in place.

### LinkedIn

Configure your own LinkedIn OAuth2 credential in n8n. Your LinkedIn Developer application must have the product/access required for creating posts, and your account must authorize the app. Select your own person/profile in the LinkedIn node after import.

The publisher export replaces the original person identifier with a placeholder. Re-select the correct person in the LinkedIn node. Do not reuse another person's identifier or credentials.

## 5. Review Workflow 1 — content generation and quality review

The supplied workflow begins with a manual trigger and an Edit Fields node containing example values for:

- `topic`
- `category`
- `audience`
- `author_experience`
- `post_type`

Edit these values to your own content topic and author profile. The prompt instructs the generator not to invent experience, statistics, sources or technical claims.

The RSS Read node is present but disabled in the supplied export. The current flow therefore uses the topic configured in Edit Fields; it does not automatically research current news.

The Content Generator returns JSON fields including `hook`, `post_content`, `hashtags`, `source_urls`, `category` and `topic`.

The AI Quality Reviewer returns JSON scores and a `decision` of `PASS` or `REWRITE`. The IF node routes PASS content to Google Sheets with `PENDING` status. A failed post goes through AI Post Rewriter, Retry Counter and Retry Limit Check. The workflow can send the rewritten content back for review; once the retry limit is reached, it appends a row with `REWRITE` status.

Run the workflow with a test topic and inspect every node's output. Check that the model returns valid JSON and that the Google Sheets expressions map the expected fields.

## 6. Review Workflow 2 — approved post publishing

The publisher is manually triggered. It:

1. Reads rows whose status is `APPROVED`.
2. Filters out rows with a non-empty `linkedin_url`.
3. Checks that status is still `APPROVED`.
4. Prepares the post and hashtags.
5. Selects one post.
6. Creates a LinkedIn post.
7. Updates the matching `post_id` row to `POSTED` and saves a URL constructed from the returned LinkedIn URN.

Before testing, use a test post that you are comfortable publishing publicly. Verify the LinkedIn account and text mapping. The publisher currently concatenates `post_content`, two newlines and `hashtags`.

The current workflow does not set a `PUBLISHING` status before the LinkedIn request. Avoid concurrent runs. A future improvement should add a lock/status transition and explicit error handling; even then, verify LinkedIn's response before retrying a failed execution to avoid duplicates.

## 7. Suggested first-run test

1. Import and configure Workflow 1.
2. Run it manually with a harmless test topic.
3. Verify a `PENDING` row appears in `Content_Queue` and inspect the content.
4. If acceptable, manually change the row status to `APPROVED`.
5. Import and configure Workflow 2.
6. Run it manually once.
7. Verify the post on your LinkedIn profile and confirm the sheet row has `POSTED` and a LinkedIn URL.

Do not run the publisher repeatedly if the LinkedIn node times out or returns an ambiguous result. First check your LinkedIn profile and the sheet.

## 8. Privacy and security

- Keep credentials in n8n's credential store.
- Do not commit `.env` files, OAuth tokens, API keys, client secrets, private spreadsheet IDs or personal data.
- Use your own accounts and obtain authorization for LinkedIn publishing.
- Use a test spreadsheet and test content during setup.
- Treat LLM review as assistance, not proof of accuracy; a human must approve content before publication.
