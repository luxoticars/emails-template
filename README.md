# Luxoticars Email Templates

Static HTML source for transactional emails sent by the dash notification Edge Functions. These files are not deployed on their own — they're uploaded into **ZeptoMail** as templates, and the dash calls ZeptoMail's send API referencing the template + merge data.

## Templates and merge tags

Each `{{tag}}` below must be added as a merge field when you create the template in ZeptoMail.

| File | Subject | Merge tags |
|---|---|---|
| `welcome.html` | Welcome to Luxoticars | `{{name}}` |
| `confirm-email.html` | Confirm your email — Luxoticars | `{{ .ConfirmationURL }}` — **Supabase Auth template, not ZeptoMail.** See note below. |
| `payment-receipt.html` | Payment confirmed — Luxoticars | `{{name}}`, `{{title}}`, `{{amount}}`, `{{currency}}`, `{{txn_ref}}`, `{{payment_date}}`, `{{renewal_date}}`, `{{url}}` |
| `post-in-review.html` | Your listing is in review — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{submitted_at}}`, `{{dash_url}}` |
| `post-published.html` | Your listing is live — Luxoticars | `{{title}}`, `{{cover_url}}`, `{{url}}` |
| `post-activated.html` | Your listing is live — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{valid_until}}`, `{{dash_url}}` |
| `post-renewed.html` | Your listing has been renewed — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{valid_until}}`, `{{dash_url}}` |
| `post-edited.html` | Your listing has been updated — Luxoticars | `{{title}}`, `{{cover_url}}`, `{{url}}` |
| `post-expiring.html` | Your listing expires soon — Luxoticars | `{{title}}`, `{{cover_url}}`, `{{days_left}}`, `{{expires_at}}`, `{{dash_url}}` |
| `post-expired.html` | Your listing has expired — Luxoticars | `{{title}}`, `{{cover_url}}`, `{{expired_at}}`, `{{dash_url}}` |
| `post-warning.html` | Action needed on your listing — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{reason}}`, `{{dash_url}}` |
| `post-rejected.html` | Your listing wasn't approved — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{reason}}`, `{{dash_url}}` |
| `post-deactivated.html` | Your listing has been taken down — Luxoticars | `{{owner_name}}`, `{{title}}`, `{{cover_url}}`, `{{dash_url}}` |

> **`confirm-email.html`** uses Supabase's Go-template syntax (`{{ .ConfirmationURL }}`), not ZeptoMail's `{{tag}}` syntax. It's the Supabase Auth "Confirm signup" email template — paste it into Supabase Dashboard → Authentication → Email Templates, not into ZeptoMail.

## Setting up a template in ZeptoMail

Do this once per file (except `confirm-email.html`, see above):

1. Log in to [ZeptoMail](https://www.zoho.com/zeptomail/) and open the **Mail Agent** used for Luxoticars transactional mail.
2. Go to **Templates → Add New Template**.
3. Give it a name that matches the file, e.g. `post-published` — the dash sends by template key/alias, so keep names consistent.
4. Switch to **HTML source / code view** and paste the full contents of the corresponding `.html` file as-is (don't edit merge tag syntax — ZeptoMail expects `{{tag_name}}`).
5. Set the **Subject** field using the `<title>` text from the table above (also written as an HTML comment/title in each file), and add any merge tags used in the subject if needed (none of these templates need dynamic subjects).
6. Save the template. ZeptoMail will auto-detect the `{{tag}}` merge fields from the HTML — verify the detected list matches the "Merge tags" column above.
7. Send a test email from the ZeptoMail preview panel, filling in sample values for every merge tag, and confirm rendering in both light and dark mode mail clients.
8. Note the **Template Key** (or alias) ZeptoMail assigns — this is what the dash's Edge Functions/Trigger.dev tasks pass in the `template_key` field of the send request. Update the corresponding environment variable / config in `luxoticars-dash` if the key differs from the filename.
9. Repeat for each template in the table.

## Updating an existing template

1. Edit the `.html` file in this repo and commit the change.
2. Copy the updated HTML into the matching ZeptoMail template's source view and save — ZeptoMail templates are not synced automatically from this repo.
3. Re-run the test send (step 7 above) to confirm merge tags still resolve correctly before considering the change live.

## Notes

- All images (logo, cover photos) are referenced by absolute URL in the HTML — there are no local/inline assets to upload separately.
- Templates use inline CSS for email-client compatibility; don't move styles into a `<style>` block without testing in Outlook/Gmail, which strip or ignore many `<style>` rules.
- `{{cover_url}}` and `{{dash_url}}` are populated by the dash from Supabase post data — leave them blank in ZeptoMail's test panel only if the template gracefully hides the image block, otherwise supply a real URL when testing.
