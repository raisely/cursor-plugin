---
description: Fundraising domain knowledge - terminology, metrics, donor models, and nonprofit compliance basics
alwaysApply: true
---

# Fundraising domain knowledge

Use this vocabulary and these concepts when reasoning about Raisely campaigns, dashboards, donor flows, and reporting. Match the user's terms; if they use a synonym, mirror it back.

## Core terminology

- **Donor**: the person giving money. Tracked as a `User` record. May be anonymous, one-off, or recurring.
- **Supporter / fundraiser**: someone running a fundraising page on behalf of the charity. Tracked as a `Profile` (individual or group/team).
- **Beneficiary**: the cause or organisation receiving funds. In Raisely, the campaign owner.
- **Gift**: a single donation. Recorded as a `Donation`.
- **Pledge**: a commitment to give later, not yet collected. Distinct from a captured donation.
- **Tribute / in-memoriam / in-honour gift**: donation made on behalf of someone else; usually triggers a notification email to a third party.
- **Matched gift / gift matching**: a sponsor commits to match donations during a window or up to a cap.
- **Soft credit**: attribution of a gift to a fundraiser or organiser other than the legal donor (e.g. company gives, employee gets soft credit).

## Fundraising models (quick recall)

- **P2P (peer-to-peer)**: supporters create profiles and ask their networks. Look at profile count, profiles activated, average raised per profile.
- **Direct donation**: donor goes straight to the charity through a form or appeal page. Look at conversion rate and average gift.
- **Events**: ticket sales + optional P2P + optional direct giving. Look at registrations, attendance, revenue per attendee.
- **DIY / community fundraising**: long-running P2P with self-serve profile creation outside a single campaign window.

See `what-is-raisely.md` for how these map to campaign themes.

## Key metrics

| Metric                             | Definition                                                                          | Why it matters                                |
| ---------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------- |
| **GTV (Gross Transaction Volume)** | Total money processed through the campaign before fees and refunds                  | Top-line fundraising result                   |
| **Net revenue**                    | GTV minus payment processing fees, platform fees, and refunds                       | What the charity actually receives            |
| **Average gift**                   | Mean donation amount over a period; report median too if the distribution is skewed | Sizing future asks and suggested amounts      |
| **Median gift**                    | Middle donation; better than average when a few large gifts distort the mean        | True "typical" donor behaviour                |
| **Conversion rate**                | Donations / unique visitors to the donate page                                      | Form quality, ask alignment, friction         |
| **Donor count**                    | Distinct donors in a period                                                         | Reach and acquisition signal                  |
| **New vs returning donor split**   | First-time donors vs repeat donors                                                  | Retention health                              |
| **Recurring donor count / MRR**    | Active recurring donors and monthly recurring revenue                               | Predictable revenue, lifetime value driver    |
| **Recurring conversion rate**      | % of donations that opt into recurring                                              | Form copy and default selection effectiveness |
| **Profiles created / activated**   | P2P only: profiles made vs profiles with at least one donation                      | Activation funnel for P2P                     |
| **Average raised per profile**     | Total P2P revenue / activated profiles                                              | Fundraiser productivity                       |
| **Cost per dollar raised (CPDR)**  | Spend / GTV                                                                         | Campaign efficiency for paid acquisition      |
| **Donor retention rate**           | % of last year's donors who gave again this year                                    | Long-term sustainability                      |
| **LTV (lifetime value)**           | Expected total giving from a donor over their lifecycle                             | Acquisition spend ceiling                     |

When a user asks "how is the campaign doing?", default to: GTV, donor count, average gift, conversion rate, and recurring split.

## One-time vs recurring giving

- **One-time gift**: single charge, captured immediately. Refundable for a window per processor rules.
- **Recurring gift / subscription**: scheduled charges (monthly is standard; weekly, fortnightly, quarterly, annual all exist). Backed by a `Subscription` record and a stored payment method.
- **Failed payment / dunning**: when a recurring charge fails, the platform retries on a schedule. Donors may need to update card details.
- **Upgrade / downgrade**: changing recurring amount. Treat as a high-value support action; never change without donor consent.
- **Cancellation**: stops future charges; does not refund prior gifts. Confirm explicitly.

Recurring donors typically have 5-10x the LTV of one-time donors. When advising on UX, suggested amounts, or copy, weigh the recurring conversion rate accordingly.

## Nonprofit compliance basics

These vary by jurisdiction. Don't give legal advice; flag when a question needs counsel. Keep these baselines in mind:

- **Tax-deductibility**: only registered charities in the donor's jurisdiction (e.g. US 501(c)(3), AU DGR, UK registered charity, CA registered charity) can issue tax-deductible receipts. Tickets, raffle entries, and goods/services received reduce the deductible amount.
- **PCI DSS**: card numbers must never touch the campaign or app server. Use Stripe Elements / hosted fields. Never log full card data, CVV, or full PANs.
- **PII and data protection**: donor names, emails, addresses, and giving history are personal data under GDPR (EU/UK), CCPA (California), and Privacy Act (AU). Honour deletion and access requests; minimise what you collect.
- **Anonymity**: "anonymous" donations should hide the donor on public pages but the charity still needs the donor record for receipting and compliance.
- **Anti-money-laundering (AML)**: large gifts (commonly $10k+ USD or local equivalent) may require source-of-funds checks. Don't auto-process unusually large gifts without flagging.
- **Solicitation registration**: in the US, charities soliciting donors in many states must register per state. The campaign disclaimer often references this.
- **Refunds**: charities generally must honour refund requests within a reasonable window. Refunding a tax-receipted gift may require revising the receipt.

## Practical heuristics

- When a user asks for a "good" suggested amount, anchor on the campaign's median gift x 1.5-2 for the middle option, not the mean.
- When optimising a donate form, recurring opt-in placement and default selection affect recurring conversion more than copy changes.
- When reporting, always pair GTV with donor count and average gift; one number alone hides the story.
- When discussing donor data, default to privacy-protective phrasing (initials, ranges, aggregates) unless the user is the charity admin and the context is internal.
