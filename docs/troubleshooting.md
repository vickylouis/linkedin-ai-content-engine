# Troubleshooting

## Workflow import

- Import the JSON into your own n8n instance using the workflow import option.
- The exports are inactive. Activate or execute only after reviewing all credentials and node settings.
- If a node type is unavailable, check your n8n version and install/update the required node package only from trusted sources.

## Groq errors

- Confirm your Groq credential is selected on every Groq Chat Model node.
- Confirm the model identifier is available to your account.
- If you hit rate or token limits, reduce prompt/input size or use an available smaller model; re-check output quality after changing models.
- If a downstream JSON parse fails, inspect the raw model output. The prompt requests JSON, but model output is not guaranteed to be valid JSON.

## Google Sheets errors

- Confirm the Google Sheets OAuth credential is connected.
- Select your own spreadsheet and `Content_Queue` tab in every Google Sheets node.
- Confirm the column names match the template exactly.
- Confirm `post_id` is populated and unique for rows that the publisher updates.
- Ensure the workflow has permission to read and append/update the sheet.

## LinkedIn errors

- Confirm your LinkedIn OAuth credential is connected and authorized.
- Confirm the LinkedIn developer app has the required posting access.
- Select the correct personal profile in the LinkedIn node.
- Check the LinkedIn node output before assuming a post was published.
- If a request times out, check your LinkedIn profile before retrying.

## Duplicate publishing risk

The supplied publisher filters for an empty `linkedin_url`, but it does not set a `PUBLISHING` lock before creating the post. Do not run concurrent publisher executions. If an execution fails after LinkedIn creates the post but before the sheet update, inspect LinkedIn and the sheet manually before retrying.

## RSS / current news

The RSS Read node is disabled in the supplied content workflow export. The current MVP uses a manually configured topic. Do not describe this version as an automated news research pipeline.
