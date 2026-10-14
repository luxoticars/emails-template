# Luxoticars Email Templates

Static HTML source for transactional emails sent by the dash notification Edge Functions. These files are not deployed on their own — they're uploaded into **ZeptoMail** as templates, and the dash calls ZeptoMail's send API referencing the template + merge data.

The `merchandise-*.html` files are archival exports for the merchandise store. They use the same brand design, but the store renders its versioned HTML and sends it directly through ZeptoMail; **do not upload these files as a required deployment step**. See [Merchandise notifications](#merchandise-notifications) below.

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

## Merchandise notifications

These four exports preserve the receipt design (navy header/footer, Luxoticars logo, blue accents, 600px table layout and inline CSS) for [luxoticars-merchandise](https://github.com/luxoticars/luxoticars-merchandise).

| File | Subject | Recipient / event |
| --- | --- | --- |
| `merchandise-receipt.html` | Payment confirmed — Luxoticars merchandise | Customer, after verified payment |
| `merchandise-admin-paid.html` | New paid order — Luxoticars merchandise | Store admin, after verified payment |
| `merchandise-fulfilled.html` | Your order is fulfilled — Luxoticars merchandise | Customer, when the owner marks the order fulfilled |
| `merchandise-admin-review.html` | Payment needs review — Luxoticars merchandise | Store admin, for late or mismatched payments |

Runtime sources are `src/lib/email-layout.ts` and `src/lib/order-email.ts` in the merchandise repository. To update these archives, edit the runtime source and run this command there with both repositories checked out as siblings:

```sh
pnpm run emails:export ../emails-template
```

Review and commit both repositories together. These files are not fetched by the live store, and changes here alone do not change outgoing mail. The exporter preserves merge tags for documentation and future reuse; the current app performs the substitutions locally and sends both HTML and plain text through ZeptoMail's `/v1.1/email` endpoint. It uses the dashboard's sender, reply-to, and credential configuration conventions. See the merchandise README for setup, queue monitoring and retry behavior.

Common fields are `{{order_id}}`, `{{event_date}}` (Malaysia time), `{{transaction_ref}}`, `{{items_rows}}`, `{{subtotal}}`, `{{delivery}}`, `{{total}}`, `{{shipping_address}}`, `{{url}}`, and `{{year}}`. Customer greetings and admin detail rows also use `{{name}}`. Admin messages additionally use `{{customer_email}}` and `{{customer_phone}}`; the review alert adds `{{review_reason}}`.

Escape all scalar fields as HTML before substitution, including attribute values. `{{items_rows}}` is trusted `<tr>` markup built from escaped item names/variants, integer quantities, unit prices and line totals; `{{shipping_address}}` consists of escaped address lines joined with `<br />`. Never place raw customer HTML in either field. Amounts include the `RM` prefix and two decimals. `{{url}}` is the configured store origin for customer messages and its authenticated `/admin` page for admin messages; do not include order access tokens.

The receipt is a payment acknowledgment, not a tax invoice. Fulfillment messages intentionally make no carrier/tracking claim because the store has no carrier integration. Payment-review alerts say “Order total”, not “Total paid”.
