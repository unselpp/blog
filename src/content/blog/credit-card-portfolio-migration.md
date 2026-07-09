---
title: "Notes on a Credit Card Portfolio Migration"
description: "How we moved a live credit card portfolio from one processor to another, and what I took away from it."
pubDate: 2026-07-08
tags: ["engineering", "payments", "migration"]
---

I had the opportunity to work on a rare kind of project: moving a live credit card portfolio from one processor to another. It touched every active cardholder, years of historical financial data, new physical cards, customer-facing flows, internal systems, external partners, and a long tail of edge cases. It was the largest and most complex project I had worked on.

A note before the story. This isn't a guide on how to migrate a credit card portfolio. It's the story of how ours went, and what I took away from it. The why behind the migration stays high-level too; that was a sound business decision and not the part worth dwelling on.

## The shape of the work

A credit card processor is the company that runs the day-to-day mechanics of a credit card program: authorizations from the network, clearings, statements, payments processing, and the ledger that tracks balances.

What stays on our side is everything between the cardholder and that infrastructure: the application flow for new cards, our app and website where cardholders see balances and make payments, the customer support experience, and the integration code that pulls information from the processor and presents it back to the user. The processor handles the financial mechanics underneath; we handle everything customers actually touch.

A migration means swapping the processor while the program keeps running. The new processor takes over new transactions. Cardholders' historical balances, payments, and statements need to come across so nothing breaks for them.

There are more stakeholders involved than you'd think. The processor you're leaving. The processor you're joining. The sponsor bank whose policies your program lives inside. The debt facility lender, with strict reporting requirements. The card network whose rails the transactions run on. And the cardholders, who should experience as little disruption as possible.

![Six external parties around one team at the center: Deserve (the processor we're leaving), Cardholders, Marqeta (the processor we're joining), Sponsor bank, Debt facility lender, and Card network, all connected to "Us."](/images/ccm-stakeholders.png)

What we did, at a high level: we built integration with the new processor while keeping the old one running. We sent each cardholder a new physical card with a new card number. When they activated it, the new processor became the source of truth for their card activity and their historical data moved across. We called the period when both cards existed, but only the old one was active, the dual-card state.

With that many moving parts, we needed a few rules going in. First, we'd choose the path that could hold up under uncertainty, even if it was not always the least amount of work. Second, when an issue surfaced, we wouldn't treat it as a one-off; we'd look at the historical data and fix every account that matched the same pattern. Third, when a decision could affect cardholders, we'd use the cardholder experience as the tiebreaker.

## The design phase

I wrote the design doc in October, working through it with Arik, our CTO. He was the person I'd bring half-formed thoughts to; he'd ask the questions that made me realize what I hadn't thought through yet. At that point the project was scoped as introducing Marqeta while running Deserve in parallel; the actual migration of existing accounts was something we knew was coming but hadn't designed for yet. The first version of the doc was mostly questions about how our existing card models would fit alongside Marqeta's.

The biggest design decision was about our data models: extend the existing ones to handle Marqeta's data, or keep them intact and build new ones. Extending risked more complexity over time. Building new ones meant two sets of logic for systems doing mostly the same thing.

I went with extending. Most of the differences were minor. Where the processor-specific logic genuinely diverged, we kept the data unified and split out the behavior cleanly.

There was one place where this didn't work cleanly: transactions. The legacy processor represented the whole lifecycle of a transaction as one record. The new processor represented each lifecycle event as its own. The data shape didn't really overlap. I'd circle back to this decision over and over in October. Eventually we kept our existing table and wrote a layer to compile Marqeta's event-per-row data into our single-row-per-transaction shape. It wasn't elegant, but it kept the existing model intact and gave us one source of truth for transaction history.

![Deserve sends one row per transaction directly into the existing transactions table; Marqeta sends N rows per transaction (authorized, cleared, disputed) through a compile layer that collapses events into one record, keeping the transactions table as the single source of truth.](/images/ccm-transaction-model.png)

The other big design call was on the customer side. A BIN is the first six digits of a card number; it tells Mastercard which issuer handles the card. We had two options. We could migrate the BIN itself, which would mean Mastercard routing traffic from our existing cards over to Marqeta after a cutoff. For cardholders this would have been frictionless; nothing would change for them. But it required Deserve, Marqeta, and Mastercard all coordinating, and we didn't know what to expect from Deserve or how fast they'd move. The other option was to use a new BIN for Marqeta, separate from Deserve's. That meant new card numbers, which meant new physical cards.

![Option A, migrate the BIN: Mastercard reroutes existing cards from Deserve to Marqeta on a cutoff — frictionless for cardholders but dependent on Deserve's pace. Option B (the one we chose), issue a new BIN: Marqeta gets its own BIN and we mail a new card to every cardholder, creating a dual-card state where both cards exist but only the old one is active until activation — no dependency on Deserve's pace, but cardholders receive a new physical card.](/images/ccm-bin-decision.png)

We went with the new BIN. More work on our end, but it didn't depend on Deserve moving at any particular pace. The risk of being blocked by a side we didn't fully control wasn't one we wanted to take. This was the simpler-solution rule with a twist: simpler didn't always mean less work; sometimes it meant the path with fewer outside dependencies.

What I remember most about October is how often I changed my mind on small things. I'd write a section, look at it for a day, see a corner case it didn't handle, and rewrite half of it. The simpler-solution rule was what kept me moving: when I couldn't tell which direction was right, I'd pick the one that would still hold up if either path won out. Looking back, that rule served me better than any of the specific design choices I made.

## Building it together

Our first migration came about four months after the design doc. Three internal accounts, three of us at the company who'd volunteered as a smoke test. By end of day, the migration platform had moved users, cards, accounts, transactions, payments, and adjustments onto Marqeta. Statements were the last step. Statements were broken.

It wasn't a bug specific to our accounts. We uncovered a statement-generation gap in the migration flow that needed to be resolved before we could continue. That fix took about a week. After that, we tested with twenty more, family-and-friends style, and uncovered another set of edge cases. Then another twenty. Then a hundred. Then a few hundred.

We knew going in that this type of migration would require close collaboration and some shared learning along the way. What we couldn't fully appreciate until we started running batches was what that would look like day-to-day. Edge cases surfaced constantly. Marqeta iterated the migration platform in real time, we iterated our data and integration alongside them. Some edges were in our data, some in their platform, and most lived in the seam between the two. It turned into a tight, fast collaboration. To Marqeta's credit, they were transparent, quick to turn around fixes, and willing to stay close to the messy details with us.

That shaped how I read every escalation. When something took a week to fix, that wasn't just a delay; it was both teams working through a new problem in real time. We started flagging things we suspected might become issues ahead of time rather than after they bit us.

![The team gathered in front of the Perpay sign in the office.](/images/ccm-team.jpg)

Alongside the engineering, the operational cadence we set up with Marqeta was just as load-bearing. Conor, our Card Program Manager, ran our side of it, and Kyle, our Product Manager, helped keep the internal pieces connected across product, operations, and engineering. Daily messages with the list of accounts being enabled. Weekly meetings that grew to three times a week and eventually a short standup most days. What mattered was that it gave both sides a place to put things they were finding, a forum where neither side could miss a developing issue, and a steady channel for sorting out who'd own what when a smart tradeoff between our side and theirs came up.

A few batches in, I'd started flagging things earlier. Not because I'd developed any clairvoyance, but because I'd been wrong enough times about timelines that I'd defaulted to assuming everything would block. A question for Marqeta's credit team that might be needed in three weeks, I'd ask now. Half the time the thing didn't end up blocking. The other half, the lead time saved us a week. The lesson wasn't strategy; it was that stakeholder responses take time, and starting the clock earlier than felt strictly necessary almost always paid off.

## After the cutover

The technical migration finished and the last cardholder moved over to Marqeta. We'd planned for a tail of edge cases after the cutover, and we got one.

The biggest pattern was balance discrepancies on a subset of migrated accounts. The data we'd brought over from Deserve had a few historical bugs baked in: the residue of fixes Deserve had made years ago whose old data still sat in their snapshot table. We applied the never-assume-one-off rule, mapped each historical bug pattern, and remediated every affected account at once. Shubham, one of the engineers on the team, partnered with me on most of that work on our side, with Marqeta helping on the fixes that touched their platform.

A few smaller classes of account-level cleanup surfaced over the months that followed. We and Marqeta worked through them together, applied the same approach, and made the affected accounts whole. That kept the customer-first rule front and center: when remediations had a cost to us, we did them anyway. The team had built that habit early, and it held.

![A Slack message from talha on Jan 20th: "A moment to celebrate 😄 All accounts have now been migrated to MQ! 🎉"](/images/ccm-migration-complete.png)

## What I'd take into the next one

Looking back, what stuck with me wasn't any specific design choice. It was that the rules we'd set at the start held up under pressure. The simpler-solution rule kept me moving in the design phase when I couldn't see ahead clearly. The never-assume-one-off rule kept us moving fast through the trailing remediation, fixing classes of issues rather than individual instances. The customer-first rule was useful most of all as a tiebreaker. When we couldn't tell what the right call was, leaning toward the cardholder gave us a consistent direction, especially during the remediation work after cutover.

Beyond the rules, the other thing I'd carry forward is everything we built around them. The early-flagging habit I picked up mid-project. The daily updates and the meetings. The daily updates and meetings were not the flashy part of the project, but they mattered. They gave us a place to put what we found, and a way to make sure issues did not get lost.

Big projects don't run on perfect plans. The plan is only one part of the system. What matters just as much is everything around it: a few clear principles for making decisions, a steady cadence for checking progress, and a repeatable way to handle surprises without losing momentum.

## Appendix

- [Marqeta — partnership announcement (LinkedIn)](https://www.linkedin.com/posts/marqeta_were-thrilled-to-announce-our-partnership-activity-7325875146869608451-XLqt/)
- [Marqeta blog — Perpay taps Marqeta to power consumer credit card](https://www.marqeta.com/blog/perpay-taps-marqeta-to-power-consumer-credit-card)
- [Partnership announcement (LinkedIn)](https://www.linkedin.com/posts/nitarlo_were-thrilled-to-announce-our-partnership-activity-7328121554121236480-a7n5/)
