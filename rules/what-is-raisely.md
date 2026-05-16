---
description: Background on what Raisely is, how campaigns work, and the fundraising models the platform supports
alwaysApply: true
---

# What is Raisely

Raisely is a fundraising platform for charities and nonprofits. Organisations use it to build and run online campaigns that collect donations, sell tickets, recruit fundraisers, and process payments end to end.

## How campaigns work

A **campaign** is the unit of fundraising on Raisely. Each campaign owns:

- A set of **pages** (home, donate, profile, leaderboard, etc.) defined as JSON and rendered by the Raisely editor and client.
- **Blocks and custom components** that compose those pages. Core blocks ship with the platform; custom components live in the campaign repo and are deployed via the Raisely CLI.
- A **theme** that determines the campaign template and visual style. The theme also signals the campaign's fundraising model.
- **Records** the campaign produces or consumes: donations, profiles (fundraising pages), users, events, tickets, subscriptions, and messages.

Campaigns are edited locally via the Raisely CLI (`raisely local`, `raisely deploy`) and previewed against live API data. The admin panel manages records and settings.

## Fundraising models

Raisely supports three core fundraising models. A single campaign picks one model via its template/theme.

### 1. Peer-to-peer (P2P)

Supporters create their own **fundraising profiles** (individual or team) under the campaign and raise money from their networks. Use this when the charity wants supporters to do the asking.

- Profiles have their own donation pages, goals, and leaderboards.
- Common templates: `charityChallenge`, `peerToPeerBasic`, `walkEvent` (P2P + event), `communityHub` (DIY fundraising).
- Records: `Profile` (individual + group), `Donation` linked to a profile, optional `Event` registration.

### 2. Direct donation

Supporters give straight to the charity through a donation form or appeal page. No fundraiser profiles are created.

- Single-page or multi-step donation flows, often with suggested amounts, recurring giving, and gift matching.
- Common templates: `donationForm`, `virtualAppeal`, `conversionAppeal`, `givingTuesday`, `givingDays*`, `inMemoriam`.
- Records: `Donation` linked directly to the campaign, `User` (donor), `Subscription` for recurring gifts.

### 3. Events

Supporters register and pay for tickets to an event (gala, walk, ride, virtual event, speaker series). Events can be ticketed-only, donation-only, or combined with P2P fundraising.

- Templates: `ticketedEvent`, `galaEvent` (donation event), `speakerEvent`, `virtualEvent`, `activeEvent`, `walkEvent` (P2P + event).
- Records: `Event`, `EventRsvp` / ticket purchases, `Donation`, and `Profile` when the event also runs P2P.

## When working with Raisely campaigns

- Treat the campaign theme as the signal for which model is in play.
- For CLI, local campaign, page JSON, and custom component work, follow `raisely-docs://guides/introduction` and read the Raisely MCP available tools first.
