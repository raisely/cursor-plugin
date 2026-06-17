---
name: raisely-create-campaign
description: Guided end-to-end workflow to create a new Raisely campaign from scratch: gather a brief, create via MCP, pull locally, customize branding/content/components, deploy, and validate. Use when the user says "Create a new campaign", "Build me a fundraising page", "Start a new Giving Tuesday campaign", "Set up a P2P campaign", "I need a new appeal page", or any similar phrase.
---

# Start a Campaign from Scratch

End-to-end workflow for creating a fully customized, branded, validated Raisely campaign — from an empty brief to a deployed page.

Run the commands yourself. Don't hand the user a checklist unless they ask.

## Before you begin

1. **Verify the MCP is connected and authenticated.** Call `get_authenticate` as a probe.
   - If it succeeds, the MCP session is valid. Continue.
   - If it returns an auth error (token expired or invalid), stop and tell the user to re-authenticate their Raisely MCP token and retry.
   - If the tool call fails entirely (MCP not reachable), stop and tell the user to configure the Raisely MCP server in their MCP client settings before proceeding. Do not continue without a working MCP session.
   - If it succeeds but `create_campaign` is unavailable (read-only session connected to `/mcp`), stop and ask the user to reconnect with `/mcp/campaigns` or send `X-Raisely-Write: campaigns`. Do not silently change the connection URL.

2. Run the `raisely-ensure-cli` skill to confirm the CLI is installed.
3. Confirm the user is logged in by running `raisely list --json`. If it returns an auth error, run `raisely login` yourself — it opens the browser for OAuth and completes automatically. Do not ask the user to run it.

---

## Step 1 — Gather the brief

If the user provides a brief document (URL, pasted text, Notion link, or file), parse it and skip the questions you can answer from it.

**If the user has not supplied a brief document**, ask before anything else:

> "Do you have a brief document for this campaign — a URL, Notion page, Google Doc, or any written description I can read?"

If they share one, parse it and use it to pre-fill as many answers as possible. If they don't have one, proceed to gather the brief through questions.

**Campaign name — ask next if not already given.** The campaign name is required before gathering the rest of the brief. If the user hasn't provided it in their message or brief, ask for it as a plain text question. Do not proceed until you have the name.

Use `AskQuestion` for every question below. Fire the first batch in a single call, then use additional `AskQuestion` calls to resolve follow-ups. Never ask for raw text when a structured choice is possible.

**Batch 1 — campaign shape and branding**

| Question | Description and recommendation | Options |
| --- | --- | --- |
| Campaign type | What kind of fundraising is this? | Appeal / P2P or teams / Event registration / Regular giving / Giving Tuesday / Other |
| Cause or sector | The area your campaign supports. | Animal welfare / Environment / Health / Humanitarian aid / Education / Community / Arts & culture / Other |
| Fundraising goal | Total amount you want to raise. | $10,000 / $25,000 / $50,000 / $100,000 / $250,000 / Other |
| Currency | The default currency for your campaign. Donors will give in this currency unless multi-currency is enabled. Recommend the currency that matches your org's country. | AUD / USD / GBP / CAD / NZD / Other |
| Donation frequencies (multi-select) | Which giving intervals can donors choose? Select all that apply. These are the only valid values — do not accept any other input. | One-time / Weekly / Fortnightly / 4 Weeks / Monthly / Every 2 months / Quarterly / Yearly |
| Preferred frequency (skip if only one frequency selected above) | Which interval is pre-selected when the donor lands on the form? Recommend "Monthly" to maximise recurring revenue. | One-time / Weekly / Fortnightly / 4 Weeks / Monthly / Every 2 months / Quarterly / Yearly |
| Multi-currency | Allow donors to give in their own currency. Recommend Yes for international campaigns; No otherwise. | Yes / No |
| Upsell to a monthly regular donation | After a one-time donation, show an extra step that encourages the donor to convert to a monthly gift. Regular donors typically make 24 donations over two years and give 4× more than one-off donors. Recommend Yes if monthly giving is enabled. | Yes / No |
| Brand colors | How will you provide your primary and secondary colors? | Extract from my website (I'll provide the URL) / I'll provide hex codes / Use theme defaults for now |
| Hero copy | How will you provide the headline and subheadline? | Generate suggestions based on my campaign / I'll write my own / Use theme defaults for now |

**Batch 2 — slug confirmation** (separate `AskQuestion` call, after deriving the slug from the campaign name)

| Question | Description and recommendation | Options |
| --- | --- | --- |
| Campaign slug | The URL path for the campaign (e.g. `sun-sun`). Derived from the campaign name — confirm or change it. | Yes, use `[derived-slug]` / No, I want a different slug (I'll provide) |

**After all batches**

For any answer that requires follow-up text ("I'll provide hex codes", "I'll write my own" copy, custom slug, or "Other"), ask for all the missing values together in one text prompt. Never send more than one text follow-up.

If the user picks "Generate suggestions based on my campaign" for hero copy, generate 2–3 headline and subheadline options and ask them to pick one via `AskQuestion` before proceeding.

If the user picks "Extract from my website" for brand colors, ask for the URL as a text follow-up, then extract the two most prominent brand colors and confirm them.

Once all fields are resolved, confirm your understanding with a one-paragraph summary and ask the user to approve before proceeding.

---

## Step 2 — Create the campaign via MCP

1. Pick the best-fit `theme` template ID based on campaign type:
   - Read the `create_campaign` MCP tool descriptor to get the current list of valid `CAMPAIGN_TEMPLATES` values.
   - Map: individual appeal → `appeal`, P2P/teams → `p2p`, event → `event`, regular giving → `regular-giving`, Giving Tuesday → `appeal`. Use `generic` when unsure.

2. Show the user the proposed values before creating:
   - Name, slug (`path`), goal in cents, currency, mode (`TEST`), theme

3. Confirm slug uniqueness: if the user has a strong preference for a slug that may already exist, note that the API will return an error and you'll surface it.

4. Call `create_campaign` with:
   - `name`: campaign display name
   - `path`: slug (URL-safe, lowercase, hyphens only)
   - `goal`: goal in **cents** (e.g. $50,000 = `5000000`)
   - `currency`: ISO 4217 code (e.g. `AUD`, `USD`, `GBP`)
   - `theme`: template ID
   - `mode`: `TEST` (always start in TEST)

5. If the call fails, surface the full API error. Common causes: slug already taken, invalid theme ID, missing required field.

6. Record the `uuid` returned. You'll use it in step 3.

---

## Step 3 — Pull locally

Before running `raisely init`, confirm the target directory. Propose `~/Raisely/campaigns/<slug>` as the default (using the campaign slug from Step 2), then use `AskQuestion` with these options:

- Use `~/Raisely/campaigns/<slug>` (default)
- I'll specify a different path

If the user picks the default, create the directory if it doesn't exist and `cd` into it. If they specify a different path, use that instead. Never run `raisely init` from an unconfirmed location.

```bash
raisely init --uuid <uuid>
```

After pulling, read the downloaded structure to understand what was scaffolded:
- List top-level files and directories.
- Note which pages and components exist.
- Check `.raisely.json` to confirm the UUID matches.

---

## Step 4 — Customize

Apply the brief in this order. Use CLI file edits and MCP calls as appropriate for each change.

### 4a — Brand (colors)

Apply these via `update_campaign_config` MCP calls:

- **Colors**: set `primaryColor` and `secondaryColor` (HEX). Raisely auto-generates shade palettes from these two values.

Image assets (logo, favicon, hero image, P2P profile image) are not supported via MCP yet. The campaign will use theme defaults. Remind the user to upload images manually via the admin UI under **Settings > Branding** when ready.

### 4b — Campaign metadata

Use `update_campaign` for:
- Goal (in cents) if it changed
- Mode stays `TEST` until the user explicitly asks to go live

### 4c — Hero copy and description

Edit the relevant page JSON files locally (pulled by `raisely init` or `raisely update` into `campaigns/<slug>/pages/`):

- **Hero headline and subheadline**: search the pulled page JSON files for the hero block (look for heading and paragraph slate nodes near the top of the `body` array on the root page). Update those text nodes with the brief values.
- **Campaign description**: search for the description or body copy block across the pulled pages and update its text nodes.
- **CTA button text**: search all pulled page JSON files for blocks with `"type": "RaiselyDonationButton"`. If any exist, use `AskQuestion` to ask the user what the button should say ("Donate now" / "Give today" / "Support us" / "Make a difference" / Other), then set the `text` field on every occurrence. If no such blocks exist, skip this step entirely — do not ask about CTA text.

### 4d — Donation form

Apply via `update_campaign_config` MCP calls:

- `defaultFrequency`: map the chosen preferred frequency to its API key: `ONCE`, `WEEK`, `2WEEK`, `4WEEK`, `MONTH`, `2MONTH`, `3MONTH`, or `YEAR`
- `regularGivingUpsell`: `true` (enable ML-powered upsell if monthly is offered)
- `multiCurrency`: `true` if the campaign is international

**Donation tiers**: the MCP `update_campaign_config` schema currently rejects array values. Instruct the user to set donation tiers manually via the admin UI under **Donation form > Amounts**.

**T&Cs checkbox and custom form fields**: no MCP or CLI path exists today. Instruct the user to add these via the admin UI under **Donation form > Custom fields**.

**Thank You message**: update the post-donation page copy in the relevant page JSON file locally, then deploy.

---

## Step 5 — Deploy and validate

Before deploying, show the user a summary:
- Pages present
- Components present
- Brand settings applied (colors, fonts)
- Goal and currency
- Dates
- Mode (TEST)
- Any items not yet applied (e.g. logo upload pending manual step)

Ask for confirmation before running deploy.

Once approved:

```bash
raisely deploy --force
```

Validation runs automatically as a pre-flight gate. If validation fails, surface the full error and fix it before re-deploying. Common issues: invalid JSON in page files, missing required config keys, component errors.

After a successful deploy, show the user the campaign URL and ask them to review it in the browser.

---

## Post-deploy reminders

Tell the user the following after every successful deploy:

**The CLI can't reach message templates.** Remind them to customize these 12 default automatic messages in the admin UI under **Settings > Messages**. At minimum:

1. **Donation Receipt** — sender name and email, subject line, branded header, PDF receipt
2. **Fundraiser Welcome** (P2P campaigns) — personalized welcome copy
3. All sender email addresses
4. Custom sending domain (Settings > Email)
5. Regional compliance copy (unsubscribe, physical address) if required

**Payments**: before going live, confirm a payment gateway (Stripe or PayPal) is connected under **Settings > Payments & Receipts**.

**Go live**: when the user is ready to accept real donations, update the campaign mode:
```
update_campaign: { mode: "LIVE" }
```
or toggle in the admin UI. Remind them this is irreversible for existing donations.

---

## Error handling reference

| Situation | Action |
| --- | --- |
| `create_campaign` fails | Surface the API error. Check slug uniqueness and theme ID validity. |
| Theme missing a needed component | Add the component during step 4e. |
| `raisely deploy` validation fails | Surface the error. Fix locally and re-deploy. |
| Payment gateway not connected | Remind the user to configure it in Settings > Payments & Receipts before going live. |
| MCP session is read-only | Ask the user to reconnect with `/mcp/campaigns` or `X-Raisely-Write: campaigns`. Do not change the URL silently. |

---

## Notes

- Always use goal values in **cents**. $50,000 = `5000000`.
- Never reference `raisely campaign download`, `raisely campaign upload`, or `raisely validate` — these commands do not exist.
- Use `raisely deploy` for upload; validation is built into it as a pre-flight gate.
- For branding and color logic, follow the same approach as the Quick Brand/Color Update skill when it exists. Until then, apply colors directly via `update_campaign_config`.
- For pre-launch checks, apply the Pre-Launch Audit skill logic when it exists. Until then, run the local preview checks in step 4f and the summary in step 5.
- `raisely start` runs against production data. Never run it here; use `raisely local` throughout.
